<p align="right">
  <a href="./README.en.md"><img src="https://img.shields.io/badge/🇺🇸-Inglês-blue?style=for-the-badge" alt="English version"></a>
  <img src="https://img.shields.io/badge/🇧🇷-Português-green?style=for-the-badge" alt="Versão em Português">
</p>

<h1 align="center">☁️ Portfólio de Projetos AWS: Cloud & Infraestrutura</h1>

<p align="center">
  <img width="2752" height="1536" alt="Portfólio de Projetos AWS" src="https://github.com/user-attachments/assets/6e09692a-e358-4310-8008-76d9230e0699" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
</p>

<p align="center">
  🇧🇷 Documentação em Português (versão em 🇺🇸 Inglês disponível acima)
</p>

<p align="center">
  Coleção de projetos práticos em Cloud Computing e Infraestrutura AWS, documentados com foco em arquitetura, segurança e raciocínio técnico, não apenas no passo a passo de execução.
</p>

---

## 📚 Sobre este repositório

Este repositório reúne os laboratórios práticos que venho desenvolvendo durante o **Programa Re/Start AWS IA + No Code**, da **Escola da Nuvem**, transformados em projetos documentados de forma profissional.

A proposta aqui vai além de simplesmente "concluir laboratórios": cada projeto documenta o problema de negócio por trás da solução técnica, as decisões de arquitetura tomadas, os desafios enfrentados e o que foi aprendido no processo. A ideia é que qualquer pessoa, técnica ou não, consiga entender não só **o que** foi feito, mas **por que** foi feito daquela forma.

Os projetos estão organizados por área de foco: armazenamento e hospedagem, segurança e controle de acesso, e arquitetura de rede.

---

## 🗂️ Projetos Documentados

### 🪣 Armazenamento e Hospedagem

**[🪣 Hospedagem de Site Estático com Amazon S3](./aws-s3-static-website-hosting)** — [PT-BR](./aws-s3-static-website-hosting/README.en.md)

Criação de um bucket Amazon S3 configurado para hospedar um site estático, com upload de arquivos via AWS CLI, políticas de acesso público controladas e um script de automação para atualizações repetíveis do site.

---

**[🗄️ Gerenciamento de Armazenamento com Amazon EBS, IAM Role e Amazon S3](./aws-storage-management)** — [PT-BR](./aws-storage-management/README.en.md)

Rotina de backup automatizado de volumes Amazon EBS via snapshots agendados, com uma IAM Role concedendo acesso seguro entre instâncias e o Amazon S3, além de sincronização de arquivos com versionamento ativo para recuperação de dados excluídos.

### 🔐 Segurança e Controle de Acesso

**[🔐 Compartilhamento Seguro de Arquivos com Amazon S3, IAM e SNS](./aws-s3-file-sharing)** — [PT-BR](./aws-s3-file-sharing/README.en.md)

Configuração de um ambiente seguro de compartilhamento de arquivos com um usuário externo, aplicando o Princípio do Menor Privilégio por meio de grupos e políticas IAM restritas, com notificações automáticas por e-mail sempre que o conteúdo do bucket é alterado.

### 🌐 Arquitetura de Rede

**[🌐 Arquitetura de Rede Segura com Amazon VPC, Bastion Host e NAT Gateway](./aws-vpc-network-foundations)** — [PT-BR](./aws-vpc-network-foundations/README.en.md)

Construção de uma rede segmentada em sub-redes públicas e privadas, com um servidor bastion controlando o acesso administrativo e um NAT Gateway garantindo conectividade de saída segura para os recursos isolados da internet.

---

**[🖥️ Rede Multi-AZ com Amazon VPC e Servidor Web em Alta Disponibilidade](./aws-vpc-web-server-deployment)** — [PT-BR](./aws-vpc-web-server-deployment/README.en.md)

Construção de uma rede distribuída em duas Zonas de Disponibilidade a partir de uma especificação de arquitetura de cliente, com um servidor web publicado e provisionado automaticamente via script de inicialização (User Data).

---

## 🧭 Como este repositório está organizado

Cada projeto possui seu próprio README, seguindo um padrão consistente:

- **Contexto e Problema:** o cenário de negócio que motivou a solução técnica.
- **Objetivo:** o que o projeto se propôs a resolver.
- **Arquitetura da Solução:** diagrama e explicação do fluxo entre os componentes.
- **O Que Foi Feito:** passo a passo real das etapas executadas.
- **Ferramentas e Serviços Utilizados:** stack técnica empregada.
- **Principais Desafios e Aprendizados:** dificuldades encontradas e o raciocínio por trás das soluções.
- **Resultado Final:** o que foi entregue ao final do projeto.

---

<a id="contact"></a>

## 📬 Contact

| Plataforma | Contato |
|:----------|:--------|
| 💼 **LinkedIn** | <a href="https://www.linkedin.com/in/lucaspimentabarretto" target="_blank">linkedin.com/in/lucaspimentabarretto</a> |
| 💻 **GitHub** | <a href="https://github.com/LucasPBar" target="_blank">github.com/LucasPBar</a> |
| 📧 **E-mail** | <a href="mailto:lucaspimenta1805@gmail.com">lucaspimenta1805@gmail.com</a> |

---

<p align="center">
  <sub>Portfólio desenvolvido como parte do <strong>Programa Re/Start AWS IA + No Code</strong>, Escola da Nuvem</sub>
</p>
