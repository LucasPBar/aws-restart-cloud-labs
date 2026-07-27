<h1 align="center">🖥️ Rede Multi-AZ com Amazon VPC e Servidor Web em Alta Disponibilidade</h1>

<p align="center">
  <img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/c11e55e7-c3aa-41ce-906b-f05ac2355bf7" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Apache-D22128?style=for-the-badge&logo=apache&logoColor=white" alt="Apache">
  <img src="https://img.shields.io/badge/status-conclu%C3%ADdo-brightgreen?style=for-the-badge" alt="Status">
</p>

<p align="center">
  Projeto prático de construção de uma rede distribuída em duas Zonas de Disponibilidade, com um servidor web público provisionado automaticamente via script de inicialização.
</p>

---

## 📚 Sobre este projeto

Este repositório documenta a construção de uma **rede AWS multi-AZ sob demanda de um cliente fictício**, incluindo o provisionamento automatizado de um servidor web público dentro dessa estrutura.

Diferente de montar cada componente manualmente peça por peça, este projeto também explora o uso do **VPC Wizard** da AWS para acelerar a criação da infraestrutura base, e um script de inicialização (User Data) para deixar o servidor pronto para uso assim que a instância sobe.

---

## 1. Contexto e Problema

Este laboratório foi realizado durante o **Programa Re/Start AWS IA + No Code**, da **Escola da Nuvem**, como parte da minha jornada de formação em Cloud Computing e Infraestrutura AWS.

O cenário parte de uma demanda real de negócio: um cliente (representado no laboratório como uma empresa Fortune 100) solicitou a construção de uma rede customizada na AWS para hospedar uma aplicação web, seguindo um diagrama de arquitetura específico fornecido por ele.

O problema central aqui não é só "subir um servidor web", mas sim **entregar exatamente a arquitetura de rede que o cliente especificou**: uma VPC distribuída em duas Zonas de Disponibilidade, com sub-redes públicas e privadas em cada uma, e um servidor web publicado na sub-rede pública da segunda zona. Esse tipo de exigência é comum em ambientes corporativos, onde a arquitetura de rede muitas vezes já vem definida por times de arquitetura ou segurança, e cabe à pessoa que implementa seguir essas especificações com precisão.

---

## 2. Objetivo

Construir a infraestrutura de rede solicitada pelo cliente e publicar um servidor web funcional dentro dela, aplicando:

- Criação de uma **VPC distribuída em múltiplas Zonas de Disponibilidade**, usando o VPC Wizard;
- Configuração de **sub-redes públicas e privadas replicadas** em duas AZs, para suportar alta disponibilidade;
- Criação de um **grupo de segurança** liberando apenas o tráfego necessário (HTTP);
- Provisionamento automatizado de um servidor web via **script de inicialização (User Data)**.

---

## 3. Arquitetura da Solução

<p align="center">
  <img width="789" height="379" alt="Image" src="https://github.com/user-attachments/assets/fd18bc02-e399-438f-b96d-1e3690a10e68" />
</p>

**Fluxo da solução:**

```
Internet
   │
   ▼
Internet Gateway
   │
   ├──▶ Zona de Disponibilidade A
   │       ├── Sub-rede pública 1 (10.0.0.0/24) ── NAT Gateway
   │       └── Sub-rede privada 1 (10.0.1.0/24)
   │
   └──▶ Zona de Disponibilidade B
           ├── Sub-rede pública 2 (10.0.2.0/24) ── Web Server 1 (Security Group: HTTP)
           └── Sub-rede privada 2 (10.0.3.0/24)
```

A VPC (`10.0.0.0/16`) foi construída com sub-redes replicadas em duas Zonas de Disponibilidade, seguindo exatamente o diagrama fornecido pelo cliente. O servidor web foi publicado na sub-rede pública da segunda AZ, protegido por um grupo de segurança que libera apenas tráfego HTTP (porta 80) vindo de qualquer origem.

---

## 4. O Que Foi Feito

Resumo das principais etapas executadas neste laboratório:

1. **Criação da estrutura de rede inicial via VPC Wizard** (opção "VPC e mais"), gerando de uma só vez: a VPC `Lab VPC` (`10.0.0.0/16`), uma sub-rede pública e uma privada na primeira Zona de Disponibilidade, um Internet Gateway, um NAT Gateway e as respectivas tabelas de rota (pública e privada).
2. **Criação manual de uma segunda sub-rede pública** (`Public Subnet 2`, `10.0.2.0/24`) e uma **segunda sub-rede privada** (`Private Subnet 2`, `10.0.3.0/24`) em uma segunda Zona de Disponibilidade, para distribuir a arquitetura entre duas AZs.
3. **Associação das novas sub-redes às tabelas de rota corretas**: a segunda sub-rede pública à tabela de rota pública, e a segunda sub-rede privada à tabela de rota privada.
4. **Criação de um grupo de segurança** (`Web Security Group`), liberando tráfego de entrada HTTP (porta 80) a partir de qualquer endereço IPv4.
5. **Lançamento da instância `Web Server 1`** na sub-rede pública da segunda Zona de Disponibilidade, com IP público automático e o grupo de segurança criado anteriormente.
6. **Configuração de um script de inicialização (User Data)** para automatizar o provisionamento do servidor:

   ```bash
   #!/bin/bash
   # Install Apache Web Server and PHP
   yum install -y httpd mysql php
   # Download Lab files
   wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RESTRT-1/267-lab-NF-build-vpc-web-server/s3/lab-app.zip
   unzip lab-app.zip -d /var/www/html/
   # Turn on web server
   chkconfig httpd on
   service httpd start
   ```

7. **Verificação da instância**, aguardando os status checks (2/2) confirmarem que ela estava saudável.
8. **Acesso ao servidor web pelo navegador**, usando o DNS público IPv4 da instância, confirmando que a aplicação estava no ar e respondendo corretamente.

---

## 5. Ferramentas e Serviços Utilizados

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Security%20Groups-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="Security Groups">
  <img src="https://img.shields.io/badge/Apache-D22128?style=for-the-badge&logo=apache&logoColor=white" alt="Apache">
  <img src="https://img.shields.io/badge/PHP-777BB4?style=for-the-badge&logo=php&logoColor=white" alt="PHP">
  <img src="https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" alt="Bash">
</p>

<br>

| Ferramenta / Serviço | Finalidade no projeto |
|---|---|
| **Amazon VPC** | Rede virtual distribuída em duas Zonas de Disponibilidade |
| **Amazon EC2** | Instância que hospeda o servidor web publicado |
| **Security Groups** | Controle de tráfego de entrada, liberando apenas HTTP |
| **Apache / PHP** | Stack utilizada para servir a aplicação web do laboratório |
| **Bash (User Data)** | Automação da instalação e configuração do servidor na inicialização da instância |

---

## 6. Principais Desafios e Aprendizados

**VPC Wizard acelera, mas não substitui entendimento da arquitetura**
Usar o VPC Wizard para criar boa parte da infraestrutura de uma vez foi bem mais rápido do que configurar cada peça manualmente, como fiz em outro laboratório de VPC. Mas isso só funcionou bem porque eu já entendia o que cada componente gerado automaticamente fazia (sub-redes, tabelas de rota, NAT Gateway). Usar um assistente sem entender o que ele está criando por trás dos panos seria arriscado em um ambiente real.

**Replicar a arquitetura em uma segunda Zona de Disponibilidade**
Ter que criar manualmente a segunda sub-rede pública e privada, e associá-las corretamente às tabelas de rota, deixou claro o motivo de arquiteturas multi-AZ existirem: distribuir recursos entre zonas fisicamente separadas é o que garante que a falha de uma única zona não derrube toda a aplicação.

**Provisionamento automatizado via User Data**
Comparado a instalar e configurar um servidor manualmente, usar um script de User Data para instalar o Apache, o PHP e baixar os arquivos da aplicação automaticamente na inicialização da instância mostrou como a AWS permite tratar servidores como algo descartável e recriável, em vez de algo configurado manualmente e mantido "à mão" para sempre.

**Seguir uma especificação de arquitetura de um cliente**
Diferente de outros laboratórios onde eu definia livremente os nomes e a estrutura, aqui existia um diagrama específico fornecido pelo "cliente" que precisava ser seguido à risca (CIDRs exatos, nomes de sub-rede, zona de disponibilidade correta). Isso se aproximou bastante de uma situação real de trabalho, onde a arquitetura muitas vezes já vem definida e a entrega precisa corresponder exatamente ao que foi especificado.

---

## 7. Resultado Final

Ao final do projeto, a infraestrutura de rede solicitada estava totalmente implementada e o servidor web estava publicamente acessível:

- ✅ VPC distribuída em duas Zonas de Disponibilidade, com sub-redes públicas e privadas replicadas em cada uma;
- ✅ Grupo de segurança liberando apenas o tráfego estritamente necessário (HTTP);
- ✅ Servidor web provisionado automaticamente via User Data, sem intervenção manual pós-lançamento;
- ✅ Aplicação acessível publicamente pelo DNS da instância, confirmando que a arquitetura solicitada pelo cliente foi entregue com sucesso.

<!--
  📌 ESPAÇO RESERVADO: PRINTS DO PAINEL AWS
  Inclua aqui capturas de tela do console AWS mostrando:
  - A VPC com as sub-redes nas duas Zonas de Disponibilidade
  - O grupo de segurança Web Security Group com a regra HTTP
  - A instância Web Server 1 com status check 2/2 passed
  - A página da aplicação web funcionando no navegador
-->

---

## 8. Competências Desenvolvidas

<p align="center">
  <img src="https://img.shields.io/badge/Cloud%20Computing-232F3E?style=flat-square&logo=amazonaws&logoColor=white" alt="Cloud Computing">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=flat-square&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=flat-square&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Alta%20Disponibilidade-8C4FFF?style=flat-square&logo=amazonaws&logoColor=white" alt="Alta Disponibilidade">
  <img src="https://img.shields.io/badge/Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white" alt="Bash">
</p>

- ☁️ Cloud Computing (AWS)
- 🌐 Amazon VPC (multi-AZ, sub-redes replicadas, tabelas de rota)
- 🖥️ Provisionamento automatizado de servidores via User Data
- 🔐 Configuração de grupos de segurança orientada à necessidade real de tráfego
- 📐 Implementação de arquitetura a partir de uma especificação de cliente
- 🛠️ Verificação e validação de disponibilidade de aplicação publicada

---

<p align="center">
  <sub>Projeto desenvolvido como parte do <strong>Programa Re/Start AWS IA + No Code</strong>, Escola da Nuvem</sub>
</p>
