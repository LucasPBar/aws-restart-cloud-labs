<h1 align="center">🪣 Hospedagem de Site Estático com Amazon S3</h1>

<p align="center">
  <img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/8b56fd35-b710-4ca3-8291-f9f926fb78e0" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/AWS-S3-orange?logo=amazons3" alt="AWS S3">
  <img src="https://img.shields.io/badge/AWS-IAM-orange?logo=amazoniam" alt="AWS IAM">
  <img src="https://img.shields.io/badge/AWS-CLI-orange?logo=amazonaws" alt="AWS CLI">
  <img src="https://img.shields.io/badge/status-conclu%C3%ADdo-brightgreen" alt="Status">
</p>

<p align="center">
  Projeto prático desenvolvido para simular a hospedagem de um site institucional simples, barato e seguro, usando o Amazon S3 como infraestrutura principal.
</p>

---

## 📚 Sobre este projeto

Este repositório documenta um laboratório prático de **hospedagem de site estático na AWS**, com foco em três pilares essenciais para qualquer profissional de Cloud/Infraestrutura: **custo**, **disponibilidade** e **segurança de acesso**.

Mais do que "subir um site no S3", o objetivo aqui foi entender **por que** cada configuração foi feita da forma como foi, documentando isso de um jeito que qualquer pessoa, técnica ou não, consiga entender o problema resolvido.

---

## 1. Contexto e Problema

Este laboratório foi realizado durante o **Programa Re/Start AWS IA + No Code**, da **Escola da Nuvem**, como parte da minha jornada de formação em Cloud Computing e Infraestrutura AWS.

O cenário proposto parte de um problema muito comum no dia a dia de pequenas empresas e times de tecnologia: **como hospedar um site simples (institucional, de divulgação ou uma landing page) de forma barata, rápida e sem precisar gerenciar um servidor inteiro?**

Manter um servidor web tradicional (EC2, por exemplo) rodando 24/7 apenas para servir um site estático (HTML, CSS, imagens) é um desperdício de recursos e de dinheiro: você paga por capacidade computacional que, na prática, mal é utilizada. Além disso, existe todo um esforço extra de manutenção, como atualizações de sistema operacional, patches de segurança e monitoramento de disponibilidade, para algo que não precisa de processamento algum, apenas entregar arquivos estáticos.

Foi justamente esse o problema que este projeto se propôs a resolver.

---

## 2. Objetivo

Implementar uma solução de **hospedagem de site estático utilizando o Amazon S3**, aplicando conceitos fundamentais de Cloud Computing como:

- Armazenamento de objetos escalável e de baixo custo;
- Controle de acesso público de forma consciente e segura;
- Uso da AWS CLI e de um usuário IAM dedicado para gerenciar recursos com boas práticas de segurança;
- Automação de tarefas repetitivas via script (deploy/atualização do site).

Além do aspecto técnico, este projeto também teve como meta desenvolver minha capacidade de **documentar decisões de arquitetura**, não só o "como fiz", mas o "por que fiz assim".

---

## 3. Arquitetura da Solução

<p align="center">
  <img width="788" height="312" alt="Image" src="https://github.com/user-attachments/assets/33d02130-0f24-4db8-8c71-e273eb2da758" />
</p>

**Fluxo da solução:**

```
Usuário (Navegador) → Endpoint de Website do Bucket S3 → Amazon S3 (Static Website Hosting)
                                                              ↓
                                              Políticas de Acesso Público + ACL
```

O bucket S3 foi configurado para atuar como servidor de arquivos estáticos, com o `index.html` definido como documento principal. O acesso público foi liberado **apenas para os objetos do site**, e não para o bucket como um todo. Essa é uma diferença importante que trato com mais detalhes na seção de aprendizados.

---

## 4. O Que Foi Feito

Resumo das principais etapas executadas neste laboratório:

1. **Conexão segura via AWS Systems Manager (Session Manager)** a uma instância EC2, sem precisar abrir portas SSH ou gerenciar chaves.
2. **Configuração da AWS CLI** na instância, usando credenciais temporárias fornecidas pelo ambiente de laboratório.
3. **Criação do bucket S3 via linha de comando** (`aws s3api create-bucket`), com nome único e região definida explicitamente.
4. **Criação de um usuário IAM dedicado** com permissões de acesso total ao Amazon S3, reforçando a prática de não usar a conta raiz/administrativa para operações do dia a dia.
5. **Ajuste das permissões do bucket**, desbloqueando o acesso público de forma controlada e habilitando ACLs para os objetos que realmente precisavam ser públicos (os arquivos do site).
6. **Extração dos arquivos do site** (pacote com `index.html`, pasta `css` e pasta de imagens) diretamente na instância EC2.
7. **Upload dos arquivos do site** para o bucket, com permissão de leitura pública configurada via ACL.
8. **Ativação da hospedagem de site estático** no bucket (`aws s3 website`), definindo o `index.html` como documento de índice.
9. **Criação de um script de atualização (`update-website.sh`)**, para tornar o processo de publicar alterações no site repetível e automatizado, em vez de repetir comandos manualmente a cada mudança.

---

## 5. Ferramentas e Serviços Utilizados

#### ☁️ Serviços AWS

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Systems%20Manager-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white" alt="AWS Systems Manager">
  <img src="https://img.shields.io/badge/AWS%20CLI-232F3E?style=for-the-badge&logo=amazonwebservices&logoColor=white" alt="AWS CLI">
</p>

#### 💻 Linguagens & Scripting

<p align="center">
  <img src="https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" alt="Bash">
  <img src="https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white" alt="HTML5">
  <img src="https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white" alt="CSS3">
</p>

<br>

| Ferramenta / Serviço | Finalidade no projeto |
|---|---|
| **Amazon S3** | Armazenamento e hospedagem do site estático |
| **AWS IAM** | Criação de usuário dedicado e controle de permissões |
| **AWS Systems Manager (Session Manager)** | Acesso seguro à instância EC2, sem exposição de portas SSH |
| **AWS CLI** | Automação da criação de recursos e deploy do site |
| **Amazon EC2** | Instância utilizada como ambiente de trabalho para execução dos comandos |
| **Bash Script** | Automação do processo de atualização do site |
| **HTML / CSS** | Conteúdo do site estático hospedado |

---

## 6. Principais Desafios e Aprendizados

**Bloqueio de acesso público x objetos realmente públicos**
Meu primeiro obstáculo foi entender que "liberar acesso público" no S3 não é uma decisão de tudo ou nada. Por padrão, a AWS bloqueia qualquer tentativa de exposição pública do bucket, o que é uma proteção correta. O aprendizado real aqui foi entender a diferença entre desbloquear o acesso público **no nível do bucket** (uma configuração necessária, mas ainda não suficiente) e conceder leitura pública **nos objetos específicos** via ACL. Essa distinção evita o erro clássico de expor um bucket inteiro quando só era necessário tornar públicos alguns arquivos.

**Versionamento como rede de segurança, não só como "recurso extra"**
Ao habilitar o versionamento, entendi na prática por que ele é considerado um requisito de confiabilidade e não apenas um item opcional: qualquer erro humano, como sobrescrever o `index.html` errado, deixa de ser um problema irreversível. Isso muda completamente a forma como se pensa em manutenção de um site "simples".

**Automatizar para reduzir erro humano**
Criar o script `update-website.sh` pareceu, a princípio, um passo trivial. Mas foi um dos aprendizados mais valiosos do laboratório: perceber que qualquer processo manual repetido mais de uma vez é candidato a virar automação, o que reduz erro humano e economiza tempo em atualizações futuras do site.

**Usuário IAM dedicado em vez da conta principal**
Criar um usuário específico para operar o S3, em vez de usar as credenciais principais do ambiente, reforçou na prática um princípio de segurança que eu já conhecia em teoria: **nunca usar credenciais de administrador para tarefas operacionais do dia a dia**, mesmo em ambiente de laboratório.

---

## 7. Resultado Final

Ao final do projeto, o site estático ficou disponível publicamente através do endpoint de website do bucket S3, seguindo o padrão:

```
http://<nome-do-bucket>.s3-website-<regiao>.amazonaws.com
```

A solução atendeu aos três requisitos centrais da missão:
- ✅ Bucket S3 configurado para hospedagem de site estático (`index.html`);
- ✅ Versionamento habilitado para proteção contra perda de dados;
- ✅ Políticas de acesso público configuradas corretamente (acesso liberado apenas onde necessário).

<!--
  📌 ESPAÇO RESERVADO: PRINTS DO PAINEL AWS
  Inclua aqui capturas de tela do console AWS mostrando:
  - O bucket criado
  - As configurações de hospedagem de site estático habilitadas
  - O versionamento ativado
  - O site funcionando no navegador
-->

---

## 8. Competências Desenvolvidas

<p align="center">
  <img src="https://img.shields.io/badge/Cloud%20Computing-232F3E?style=flat-square&logo=amazonaws&logoColor=white" alt="Cloud Computing">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=flat-square&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=flat-square&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/AWS%20CLI-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white" alt="AWS CLI">
  <img src="https://img.shields.io/badge/Troubleshooting-4B5563?style=flat-square&logo=probot&logoColor=white" alt="Troubleshooting">
  <img src="https://img.shields.io/badge/Documenta%C3%A7%C3%A3o%20T%C3%A9cnica-4B5563?style=flat-square&logo=readthedocs&logoColor=white" alt="Documentação Técnica">
</p>

- ☁️ Cloud Computing (AWS)
- 🪣 Amazon S3 (Static Website Hosting, ACLs, Versionamento)
- 🔐 AWS IAM (criação de usuários, políticas gerenciadas)
- 💻 AWS CLI e automação via Bash
- 🛠️ Troubleshooting de permissões e acesso público
- 📝 Documentação técnica orientada a negócio

---

<p align="center">
  <sub>Projeto desenvolvido como parte do <strong>Programa Re/Start AWS IA + No Code</strong>, Escola da Nuvem</sub>
</p>
