<h1 align="center">🔐 Compartilhamento Seguro de Arquivos com Amazon S3, IAM e SNS</h1>

<p align="center">
  <img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/631ffb97-70fb-49d4-9651-fe28e76dd65c" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/Amazon%20SNS-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon SNS">
  <img src="https://img.shields.io/badge/status-conclu%C3%ADdo-brightgreen?style=for-the-badge" alt="Status">
</p>

<p align="center">
  Projeto prático de configuração de um ambiente seguro para compartilhamento de arquivos com um usuário externo, aplicando o Princípio do Menor Privilégio e notificações automáticas de auditoria.
</p>

---

## 📚 Sobre este projeto

Este repositório documenta a configuração de um **bucket Amazon S3 compartilhado com um usuário externo**, simulando um cenário real de negócio: uma cafeteria fictícia contrata uma empresa de mídia para gerenciar as fotos dos produtos vendidos no site.

O foco central do projeto não é só "dar acesso a alguém", mas sim **dar o acesso certo, na medida certa**, e ainda manter visibilidade sobre tudo o que é alterado no ambiente.

---

## 1. Contexto e Problema

Este laboratório foi realizado durante o **Programa Re/Start AWS IA + No Code**, da **Escola da Nuvem**, como parte da minha jornada de formação em Cloud Computing e Infraestrutura AWS.

O cenário parte de um problema comum quando empresas precisam colaborar com fornecedores ou parceiros externos: **como conceder acesso a um recurso da nuvem para alguém de fora da empresa, sem expor todo o ambiente e sem perder o controle sobre o que essa pessoa faz?**

No caso deste projeto, uma cafeteria fictícia contratou uma empresa de mídia para tirar fotos dos produtos e atualizar as imagens usadas no site. Essa pessoa externa (representada pelo usuário `mediacouser`) precisa conseguir enviar, atualizar e remover imagens em um bucket S3, mas **não deve ter acesso a mais nada no ambiente AWS**, muito menos permissão para alterar configurações de segurança do próprio bucket.

Além disso, a equipe da cafeteria precisa saber, em tempo real, sempre que o conteúdo do bucket for alterado, sem precisar ficar checando o console manualmente.

---

## 2. Objetivo

Configurar um ambiente seguro de compartilhamento de arquivos utilizando Amazon S3, aplicando conceitos de:

- Controle de acesso granular via **grupos e usuários IAM**;
- Aplicação prática do **Princípio do Menor Privilégio**;
- Automação de comandos via **AWS CLI**;
- **Notificações automáticas** de eventos do bucket via Amazon SNS.

Mais do que configurar as permissões, o objetivo foi testar e comprovar na prática que elas realmente funcionam como esperado, tanto para o que deveria ser permitido quanto para o que deveria ser bloqueado.

---

## 🔐 Princípio do Menor Privilégio (Least Privilege)

O **Princípio do Menor Privilégio** é um dos pilares mais importantes de segurança em nuvem. Ele estabelece que **qualquer usuário, aplicação ou serviço deve receber apenas as permissões estritamente necessárias para realizar sua função, e nada além disso**.

Na prática, isso significa que, em vez de conceder acesso amplo "por conveniência", cada permissão é pensada e justificada individualmente: *por que esse usuário precisa dessa ação, nesse recurso específico?*

**Por que esse princípio é importante:**

- **Reduz a superfície de ataque**: quanto menos permissões um usuário ou credencial tem, menor o dano possível caso essas credenciais sejam comprometidas.
- **Limita o "raio de explosão" de um erro**: um usuário mal configurado ou uma automação com bug não consegue afetar recursos fora do seu escopo de permissão.
- **Facilita auditoria e conformidade**: permissões específicas e documentadas tornam muito mais simples entender "quem pode fazer o quê" em um ambiente.
- **Evita acúmulo de privilégios ao longo do tempo**, um problema comum em ambientes reais onde usuários vão recebendo acessos extras e nunca os perdem.

**Como esse princípio foi aplicado neste laboratório:**

O usuário `mediacouser` está associado a um grupo IAM (`mediaco`) com uma política customizada (`mediaCoPolicy`) que concede **apenas três tipos de permissão**, todas restritas a um prefixo específico do bucket (`cafe-*/images/*`):

- Listar o bucket no console;
- Ler, adicionar e remover objetos dentro da pasta `images/`.

Esse usuário **não tem permissão para alterar políticas do bucket, ajustar permissões públicas ou acessar qualquer outro recurso da conta**. Essa restrição foi validada na prática durante o laboratório (veja a seção de aprendizados).

---

## 3. Arquitetura da Solução

<p align="center">
  <img width="770" height="482" alt="Image" src="https://github.com/user-attachments/assets/6fdb5793-c24a-4c4f-9842-99370d50d9f8" />
</p>

**Fluxo da solução:**

```
mediacouser (Console ou AWS CLI)
        │
        ▼
Amazon S3 (Cafe S3 bucket)
        │
        ▼  (evento de criação/remoção de objeto)
Amazon SNS (s3NotificationTopic)
        │
        ▼
Administrador (recebe notificação por e-mail)
```

O usuário externo `mediacouser` interage com o bucket S3 apenas dentro do escopo permitido pela política do grupo `mediaco`. Toda vez que um objeto é criado ou removido do bucket, o S3 publica um evento no tópico SNS `s3NotificationTopic`, que envia automaticamente um e-mail de notificação para o administrador.

---

## 4. O Que Foi Feito

Resumo das principais etapas executadas neste laboratório:

1. **Conexão à instância CLI Host via EC2 Instance Connect** e configuração da AWS CLI com as credenciais fornecidas pelo ambiente.
2. **Criação do bucket S3** (`cafe-xxxnnn`) via linha de comando (`aws s3 mb`).
3. **Upload inicial de imagens** para o bucket, dentro do prefixo `/images`, usando `aws s3 sync`.
4. **Revisão do grupo IAM `mediaco`** e da política `mediaCoPolicy`, entendendo cada uma das três permissões concedidas (listar bucket, listar objetos, e ler/gravar/excluir apenas no prefixo `images/`).
5. **Revisão do usuário IAM `mediacouser`**, confirmando que ele herda as permissões do grupo, e criação de uma access key para uso via CLI.
6. **Teste de acesso como `mediacouser`** pelo Console AWS: visualizar uma imagem, fazer upload de uma nova imagem e excluir uma imagem existente, todas ações permitidas com sucesso.
7. **Teste de uma ação não autorizada**: tentativa de alterar as permissões do bucket como `mediacouser`, resultando em erro de acesso negado, confirmando que a política estava corretamente restritiva.
8. **Criação do tópico SNS `s3NotificationTopic`**, com política de acesso permitindo que o S3 publique mensagens nele.
9. **Assinatura do tópico por e-mail** e confirmação da inscrição.
10. **Configuração de notificação de eventos no bucket** (`ObjectCreated` e `ObjectRemoved`, restrita ao prefixo `images/`), associada ao tópico SNS via arquivo JSON e AWS CLI.
11. **Testes via AWS CLI usando as credenciais do `mediacouser`**: upload de objeto (gerou notificação), leitura de objeto (não gerou notificação, como esperado) e exclusão de objeto (gerou notificação).
12. **Novo teste de ação não autorizada via CLI**: tentativa de tornar um objeto público via `put-object-acl`, bloqueada com erro `AccessDenied`.

---

## 5. Ferramentas e Serviços Utilizados

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/Amazon%20SNS-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon SNS">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/AWS%20CLI-232F3E?style=for-the-badge&logo=amazonwebservices&logoColor=white" alt="AWS CLI">
</p>

<br>

| Ferramenta / Serviço | Finalidade no projeto |
|---|---|
| **Amazon S3** | Armazenamento e compartilhamento das imagens dos produtos |
| **AWS IAM** | Grupo e usuário dedicados ao acesso externo, com permissões restritas |
| **Amazon SNS** | Envio de notificações por e-mail sobre alterações no bucket |
| **Amazon EC2 (EC2 Instance Connect)** | Acesso à instância CLI Host usada para executar os comandos |
| **AWS CLI** | Criação do bucket, upload de arquivos e configuração de notificações |

---

## 6. Principais Desafios e Aprendizados

**Entender permissões por prefixo, não só por bucket**
Um dos pontos mais interessantes da política `mediaCoPolicy` foi perceber que ela concede acesso apenas a objetos dentro de `cafe-*/images/*`, e não ao bucket inteiro. Isso me fez repensar como estruturar buckets no futuro: organizar o conteúdo por prefixos desde o início facilita muito a aplicação de permissões granulares depois.

**Confirmar restrições na prática, não só na teoria**
Ler uma política JSON e entender o que ela permite é uma coisa. Testar de fato, logado como o próprio usuário restrito, tentando fazer algo fora do escopo e receber o erro de acesso negado, é outra completamente diferente. Esse teste prático deu muito mais confiança de que a configuração estava correta do que apenas revisar o texto da política.

**Notificações como camada de visibilidade, não de controle**
Configurar o SNS deixou claro que notificação de evento não substitui controle de acesso, ela complementa. Mesmo com as permissões corretamente restritas, ter um e-mail automático sempre que um arquivo é criado ou removido dá uma camada extra de rastreabilidade sobre o que está acontecendo no ambiente, útil tanto para auditoria quanto para detectar comportamentos inesperados.

**Diferença entre ações que geram evento e ações que não geram**
Achei relevante notar que a leitura de um objeto (`get-object`) não dispara notificação, apenas criação e remoção. Isso reforça que o sistema de notificação foi desenhado para eventos de mudança de estado, e não para monitorar todo tipo de acesso, o que também tem implicações se algum dia for necessário auditar leituras.

---

## 7. Resultado Final

Ao final do projeto, o ambiente estava configurado com:

- ✅ Bucket S3 dedicado ao compartilhamento de imagens com um usuário externo;
- ✅ Grupo e usuário IAM com permissões restritas ao prefixo `images/`, validados na prática (inclusive testando o que **não** deveria funcionar);
- ✅ Notificações automáticas por e-mail para o administrador sempre que um objeto é criado ou removido.

<!--
  📌 ESPAÇO RESERVADO: PRINTS DO PAINEL AWS
  Inclua aqui capturas de tela do console AWS mostrando:
  - A política mediaCoPolicy expandida no IAM
  - O teste de acesso negado ao tentar alterar permissões do bucket como mediacouser
  - O tópico SNS criado e a assinatura confirmada
  - Um e-mail de notificação recebido
-->

---

## 8. Competências Desenvolvidas

<p align="center">
  <img src="https://img.shields.io/badge/Cloud%20Computing-232F3E?style=flat-square&logo=amazonaws&logoColor=white" alt="Cloud Computing">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=flat-square&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=flat-square&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/Amazon%20SNS-DD344C?style=flat-square&logo=amazonaws&logoColor=white" alt="Amazon SNS">
  <img src="https://img.shields.io/badge/AWS%20CLI-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white" alt="AWS CLI">
</p>

- ☁️ Cloud Computing (AWS)
- 🔐 AWS IAM (grupos, usuários, políticas customizadas e Princípio do Menor Privilégio)
- 🪣 Amazon S3 (permissões por prefixo, gerenciamento via CLI)
- 📩 Amazon SNS (tópicos, assinaturas, notificação de eventos)
- 💻 AWS CLI e automação de tarefas
- 🛠️ Validação prática de controles de segurança (testes de acesso autorizado e não autorizado)

---

<p align="center">
  <sub>Projeto desenvolvido como parte do <strong>Programa Re/Start AWS IA + No Code</strong>, Escola da Nuvem</sub>
</p>
