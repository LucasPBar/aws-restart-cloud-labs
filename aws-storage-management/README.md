<h1 align="center">🗄️ Gerenciamento de Armazenamento com Amazon EBS, IAM Role e Amazon S3</h1>

<p align="center">
  <img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/bb4ec8fd-080b-4b2e-9aaa-cbf3377e11a2" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Amazon%20EBS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon EBS">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/status-conclu%C3%ADdo-brightgreen?style=for-the-badge" alt="Status">
</p>

<p align="center">
  Projeto prático de backup automatizado de volumes EBS, com controle de acesso baseado em Role IAM e recuperação de arquivos via versionamento no Amazon S3.
</p>

---

## 📚 Sobre este projeto

Este repositório documenta a criação de uma rotina de **backup automatizado de dados em disco (Amazon EBS)**, incluindo a limpeza de snapshots antigos e a sincronização de arquivos com o Amazon S3 como camada extra de proteção.

O projeto também aborda um ponto central de segurança em nuvem: como conceder a uma instância EC2 acesso a outros serviços da AWS **sem usar credenciais fixas**, através de uma IAM Role.

---

## 1. Contexto e Problema

Este laboratório foi realizado durante o **Programa Re/Start AWS IA + No Code**, da **Escola da Nuvem**, como parte da minha jornada de formação em Cloud Computing e Infraestrutura AWS.

O cenário parte de um problema muito comum em ambientes de produção: **como proteger os dados armazenados em um servidor contra perda acidental, sem depender de processos manuais de backup que alguém pode esquecer de executar?**

Além disso, existe um segundo problema, este relacionado à segurança: quando uma instância EC2 precisa acessar outros serviços da AWS (como o S3), qual é a forma correta de conceder essa permissão? Embutir chaves de acesso fixas dentro da instância é uma prática arriscada, caso essas credenciais vazem, o dano pode ser bem maior do que o esperado.

Este projeto resolve os dois problemas ao mesmo tempo: cria uma rotina de snapshots automatizada para o volume EBS, e usa uma IAM Role para dar à instância acesso ao S3 de forma seguindo boas práticas de segurança.

---

## 2. Objetivo

Implementar uma rotina de gerenciamento de armazenamento na AWS, cobrindo:

- Criação e manutenção automatizada de **snapshots de volumes EBS**;
- Concessão de permissões a uma instância EC2 via **IAM Role**, em vez de credenciais fixas;
- Sincronização de arquivos entre um volume EBS e um **bucket S3**;
- Uso do **versionamento do S3** como mecanismo de recuperação de arquivos excluídos por engano.

---

## 🔐 Princípio do Menor Privilégio (Least Privilege)

O **Princípio do Menor Privilégio** estabelece que qualquer usuário, aplicação ou recurso deve ter acesso apenas ao que é estritamente necessário para cumprir sua função, nada além disso.

**Por que esse princípio é importante:**

- **Reduz a superfície de ataque**: se uma credencial ou recurso é comprometido, o estrago possível fica limitado ao escopo de permissões que ele realmente tem.
- **Evita acoplamento desnecessário entre serviços**: uma instância que só precisa acessar o S3 não deveria ter permissão para, por exemplo, apagar bancos de dados ou gerenciar usuários.
- **Facilita a auditoria**: permissões específicas tornam claro o motivo pelo qual cada acesso existe, em vez de depender de "acesso total só para garantir que funcione".

**Como esse princípio foi aplicado neste laboratório:**

Em vez de armazenar uma chave de acesso da AWS diretamente na instância "Processor" (o que exigiria gerenciar, rotacionar e proteger essa credencial manualmente), o laboratório usa uma **IAM Role** (`S3BucketAccess`) anexada como *instance profile* à instância. Essa Role concede exatamente as permissões necessárias para a instância interagir com volumes EBS e com o bucket S3 do laboratório, e nada além disso.

Essa abordagem é considerada uma boa prática de segurança na AWS: **Roles eliminam a necessidade de credenciais fixas em instâncias**, já que a AWS gerencia automaticamente a rotação das credenciais temporárias por trás da Role.

---

## 3. Arquitetura da Solução

<p align="center">
  <img width="778" height="486" alt="Image" src="https://github.com/user-attachments/assets/0bdd8e62-9bf5-4241-a8cf-c8121b7df442" />
</p>

**Fluxo da solução:**

```
Command Host  ──(administra via AWS CLI)──▶  Processor (IAM Role: S3BucketAccess)
                                                    │
                                                    ▼
                                             Volume EBS
                                              │        │
                                              ▼        ▼
                                         Snapshot   S3 sync ──▶ Bucket S3 (versionado)
```

A instância "Command Host" é usada para administrar o ambiente via AWS CLI. A instância "Processor" possui a Role `S3BucketAccess` anexada, o que permite que ela (e os comandos executados nela) acessem o volume EBS e o bucket S3 sem precisar de credenciais fixas. A partir daí, o fluxo se divide em duas frentes de proteção de dados: snapshots do volume EBS e sincronização de arquivos com o S3, este último com versionamento ativado.

---

## 4. O Que Foi Feito

Resumo das principais etapas executadas neste laboratório:

1. **Criação de um bucket S3**, que serviu de destino para a sincronização de arquivos do volume EBS.
2. **Anexação da IAM Role `S3BucketAccess`** como instance profile na instância EC2 "Processor", concedendo a ela permissão de interagir com o EBS e o S3.
3. **Conexão à instância "Command Host"** via EC2 Instance Connect, usada para administrar o ambiente.
4. **Identificação do volume EBS e do ID da instância "Processor"** via consultas com `aws ec2 describe-instances`.
5. **Parada da instância "Processor"** antes da criação do snapshot inicial, para garantir consistência dos dados.
6. **Criação do primeiro snapshot do volume EBS** (`aws ec2 create-snapshot`) e verificação da conclusão do processo.
7. **Reinicialização da instância "Processor"** após o snapshot ser concluído.
8. **Criação de um cron job** para gerar um novo snapshot automaticamente a cada minuto, simulando uma rotina de backup recorrente.
9. **Execução do script Python `snapshotter_v2.py`**, que identifica todos os snapshots do volume e mantém apenas os dois mais recentes, removendo os demais automaticamente.
10. **Desafio opcional: sincronização com o Amazon S3**, incluindo:
    - Ativação do versionamento no bucket S3 (`aws s3api put-bucket-versioning`);
    - Sincronização de uma pasta local com o bucket via `aws s3 sync`;
    - Exclusão de um arquivo local e propagação dessa exclusão para o S3 usando `aws s3 sync --delete`;
    - Consulta ao histórico de versões do arquivo excluído (`aws s3api list-object-versions`);
    - Recuperação da versão anterior do arquivo (`aws s3api get-object --version-id`) e nova sincronização para restaurá-lo no bucket.

---

## 5. Ferramentas e Serviços Utilizados

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Amazon%20EBS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon EBS">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/AWS%20CLI-232F3E?style=for-the-badge&logo=amazonwebservices&logoColor=white" alt="AWS CLI">
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/Cron-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" alt="Cron">
</p>

<br>

| Ferramenta / Serviço | Finalidade no projeto |
|---|---|
| **Amazon EC2** | Instâncias "Command Host" (administração) e "Processor" (recurso protegido) |
| **Amazon EBS** | Volume de armazenamento em disco que recebeu os snapshots |
| **Amazon S3** | Destino da sincronização de arquivos e camada extra de backup com versionamento |
| **AWS IAM (Role)** | Concessão de permissões à instância "Processor" sem uso de credenciais fixas |
| **AWS CLI** | Execução de todos os comandos de snapshot, sync e configuração |
| **Python** | Script de retenção automática dos dois snapshots mais recentes |
| **Cron** | Agendamento da criação recorrente de snapshots |

---

## 6. Principais Desafios e Aprendizados

**Role em vez de chave de acesso fixa**
Antes deste laboratório, eu associava "dar acesso a uma instância" apenas à ideia de configurar `aws configure` com uma chave de acesso. Ver a Role `S3BucketAccess` sendo anexada diretamente como instance profile, sem nenhuma credencial visível ou copiada manualmente, mudou minha forma de pensar sobre como instâncias deveriam acessar outros serviços da AWS.

**Por que parar a instância antes do snapshot**
Percebi que o laboratório para a instância "Processor" antes de tirar o primeiro snapshot. Isso reforçou um conceito importante de consistência de dados: tirar um snapshot com a instância em execução pode capturar o disco em um estado intermediário (com escritas pendentes em cache), enquanto parar a instância garante que tudo que precisa estar gravado no volume, esteja.

**Automação de retenção com o script Python**
Rodar o `snapshotter_v2.py` e ver a lógica de manter apenas os dois snapshots mais recentes me fez perceber um problema real de operação: snapshots automatizados sem uma política de retenção geram custo crescente indefinidamente. Automatizar não é só criar backups, é também gerenciar o ciclo de vida deles.

**Versionamento como rede de segurança real, testada na prática**
Diferente de apenas habilitar o versionamento e seguir em frente, este laboratório me fez de fato excluir um arquivo, confirmar que ele sumiu do bucket, e então recuperá-lo usando o histórico de versões. Isso deixou muito mais claro, na prática, por que versionamento é considerado um requisito de confiabilidade e não apenas uma configuração opcional.

---

## 7. Resultado Final

Ao final do projeto, o ambiente contava com:

- ✅ Rotina de snapshots automatizada para o volume EBS, com retenção controlada dos dois snapshots mais recentes;
- ✅ Instância EC2 acessando o S3 por meio de uma IAM Role, sem uso de credenciais fixas;
- ✅ Sincronização de arquivos com o Amazon S3, incluindo versionamento ativo e recuperação bem-sucedida de um arquivo excluído.

<!--
  📌 ESPAÇO RESERVADO: PRINTS DO PAINEL AWS
  Inclua aqui capturas de tela do console AWS mostrando:
  - A Role S3BucketAccess anexada à instância Processor
  - A lista de snapshots antes e depois da execução do script Python
  - O versionamento habilitado no bucket S3
  - O histórico de versões do arquivo recuperado
-->

---

## 8. Competências Desenvolvidas

<p align="center">
  <img src="https://img.shields.io/badge/Cloud%20Computing-232F3E?style=flat-square&logo=amazonaws&logoColor=white" alt="Cloud Computing">
  <img src="https://img.shields.io/badge/Amazon%20EBS-FF9900?style=flat-square&logo=amazonaws&logoColor=white" alt="Amazon EBS">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=flat-square&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=flat-square&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/AWS%20CLI-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white" alt="AWS CLI">
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python">
</p>

- ☁️ Cloud Computing (AWS)
- 💾 Amazon EBS (snapshots, backup e restauração)
- 🔐 AWS IAM (Roles e Princípio do Menor Privilégio aplicado a recursos de computação)
- 🪣 Amazon S3 (sincronização de arquivos e versionamento)
- 💻 AWS CLI e automação via script Python e cron
- 🛠️ Gerenciamento de ciclo de vida de backups

---

<p align="center">
  <sub>Projeto desenvolvido como parte do <strong>Programa Re/Start AWS IA + No Code</strong>, Escola da Nuvem</sub>
</p>
