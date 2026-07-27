<h1 align="center">🌐 Arquitetura de Rede Segura com Amazon VPC, Bastion Host e NAT Gateway</h1>

<p align="center">
  <img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/ab2b2d9f-0ca6-45b7-b48d-43c0edded487" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/NAT%20Gateway-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="NAT Gateway">
  <img src="https://img.shields.io/badge/status-conclu%C3%ADdo-brightgreen?style=for-the-badge" alt="Status">
</p>

<p align="center">
  Projeto prático de construção de uma rede privada isolada na AWS, com sub-redes pública e privada, acesso controlado via servidor bastion e saída para internet protegida por NAT Gateway.
</p>

---

## 📚 Sobre este projeto

Este repositório documenta a construção de uma **arquitetura de rede segmentada na AWS**, separando recursos que precisam ser acessíveis pela internet daqueles que devem ficar isolados, mas ainda assim precisam de conectividade de saída.

O projeto simula um cenário muito comum em ambientes corporativos: como estruturar uma rede em nuvem onde apenas o estritamente necessário fica exposto publicamente, e o restante permanece protegido, mas ainda operacional.

---

## 1. Contexto e Problema

Este laboratório foi realizado durante o **Programa Re/Start AWS IA + No Code**, da **Escola da Nuvem**, como parte da minha jornada de formação em Cloud Computing e Infraestrutura AWS.

O cenário parte de um problema clássico de arquitetura de redes na nuvem: **como projetar um ambiente onde alguns recursos precisam ser acessados diretamente pela internet, enquanto outros (como bancos de dados ou servidores internos) devem ficar completamente inacessíveis externamente, mas ainda assim conseguir se atualizar e se comunicar com serviços externos quando necessário?**

Expor todos os recursos diretamente à internet é uma prática arriscada: qualquer instância publicamente acessível se torna um alvo potencial. Por outro lado, isolar completamente um recurso sem nenhuma saída para a internet também é inviável na prática, já que servidores internos frequentemente precisam baixar atualizações, pacotes ou se comunicar com APIs externas.

Este projeto resolve esse dilema usando um padrão de arquitetura amplamente adotado na AWS: segmentação em sub-redes públicas e privadas, com um servidor bastion controlando o acesso administrativo e um NAT Gateway controlando a saída de tráfego da sub-rede privada.

---

## 2. Objetivo

Construir uma VPC funcional, aplicando conceitos fundamentais de arquitetura de rede em nuvem:

- Separação de recursos em **sub-redes públicas e privadas**;
- Controle de acesso administrativo via **servidor bastion** (jump box);
- Conectividade de saída segura em sub-redes privadas via **NAT Gateway**;
- Configuração de **tabelas de rota** para direcionar corretamente o tráfego local, público e privado.

---

## 3. Arquitetura da Solução

<p align="center">
  <img width="750" height="471" alt="Image" src="https://github.com/user-attachments/assets/5257098f-0185-4579-8afc-4bcf94d47895" />
</p>

**Fluxo da solução:**

```
Internet
   │
   ▼
Internet Gateway
   │
   ▼
Sub-rede pública (10.0.0.0/24)
   ├── Bastion Host  ◀── acesso administrativo (SSH)
   └── NAT Gateway
             │
             ▼
Sub-rede privada (10.0.2.0/23)
   └── Instância privada  ── acessa a internet apenas de saída, via NAT Gateway
```

A VPC (`10.0.0.0/16`) foi dividida em uma sub-rede pública e uma sub-rede privada. A sub-rede pública recebe tráfego direto da internet através de um Internet Gateway, e é onde ficam o servidor bastion e o NAT Gateway. A sub-rede privada não tem rota direta para a internet: todo o tráfego de saída passa obrigatoriamente pelo NAT Gateway, e o único caminho de acesso administrativo é através do servidor bastion.

---

## 4. O Que Foi Feito

Resumo das principais etapas executadas neste laboratório:

1. **Criação da VPC "Lab VPC"** com bloco CIDR `10.0.0.0/16` e nomes de host DNS habilitados.
2. **Criação de uma sub-rede pública** (`10.0.0.0/24`), configurada para atribuir automaticamente IPs públicos às instâncias lançadas nela.
3. **Criação de uma sub-rede privada** (`10.0.2.0/23`), sem atribuição automática de IP público.
4. **Criação e anexação de um Internet Gateway** (`Lab IGW`) à VPC, permitindo comunicação entre a sub-rede pública e a internet.
5. **Configuração das tabelas de rota**: uma tabela pública com rota `0.0.0.0/0` apontando para o Internet Gateway (associada à sub-rede pública), e uma tabela privada, inicialmente apenas com a rota local.
6. **Lançamento do servidor bastion** (`Bastion Server`) na sub-rede pública, com grupo de segurança liberando acesso SSH.
7. **Criação de um NAT Gateway** na sub-rede pública, com um IP elástico alocado.
8. **Atualização da tabela de rota privada**, adicionando uma rota `0.0.0.0/0` apontando para o NAT Gateway, dando à sub-rede privada acesso de saída à internet.
9. **Desafio opcional: lançamento de uma instância privada**, com grupo de segurança permitindo SSH apenas a partir do intervalo interno da VPC (`10.0.0.0/16`), ou seja, apenas do bastion.
10. **Login na instância privada através do servidor bastion**, conectando primeiro ao bastion via EC2 Instance Connect e, a partir dele, via SSH até a instância privada.
11. **Teste de conectividade com a internet a partir da instância privada** usando `ping`, confirmando que o tráfego de saída estava passando corretamente pelo NAT Gateway.

---

## 5. Ferramentas e Serviços Utilizados

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Internet%20Gateway-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Internet Gateway">
  <img src="https://img.shields.io/badge/NAT%20Gateway-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="NAT Gateway">
  <img src="https://img.shields.io/badge/Security%20Groups-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="Security Groups">
</p>

<br>

| Ferramenta / Serviço | Finalidade no projeto |
|---|---|
| **Amazon VPC** | Rede virtual isolada, com sub-redes pública e privada |
| **Internet Gateway** | Conectividade bidirecional entre a sub-rede pública e a internet |
| **NAT Gateway** | Conectividade de saída para a sub-rede privada, sem exposição a conexões de entrada |
| **Amazon EC2** | Servidor bastion (acesso administrativo) e instância privada (recurso protegido) |
| **Security Groups** | Controle de tráfego de entrada em nível de instância (SSH liberado apenas de origens específicas) |
| **Route Tables** | Direcionamento do tráfego local, público e privado dentro da VPC |

---

## 6. Principais Desafios e Aprendizados

**O nome da sub-rede não define se ela é pública**
Um dos aprendizados mais importantes deste laboratório foi perceber que chamar uma sub-rede de "pública" não a torna pública de fato. O que realmente determina isso é a **tabela de rota associada a ela**: só depois de associar a sub-rede a uma tabela com rota para o Internet Gateway ela passa a ter conectividade real com a internet.

**Por que usar um servidor bastion em vez de expor tudo diretamente**
Entender o padrão de bastion host (jump box) deixou claro por que ele é tão usado em arquiteturas reais: em vez de expor cada recurso interno diretamente à internet, existe um único ponto de entrada controlado e monitorado. Se esse ponto único de acesso for bem protegido, toda a rede interna fica mais segura.

**A diferença entre Internet Gateway e NAT Gateway**
No início, os dois conceitos pareciam parecidos, mas o laboratório deixou a diferença bem clara na prática: o Internet Gateway permite tráfego de entrada e saída para recursos com IP público (como o bastion), enquanto o NAT Gateway permite apenas tráfego de saída, sem expor a sub-rede privada a conexões iniciadas de fora. É uma diferença pequena na explicação, mas enorme em termos de segurança.

**Validar a configuração com um teste real, não só assumir que funcionou**
Executar o `ping` a partir da instância privada para confirmar a saída para a internet reforçou um hábito importante: depois de configurar uma rede, é fundamental testar a conectividade de ponta a ponta, em vez de assumir que a configuração está correta só porque não apareceu nenhum erro no console.

---

## 7. Resultado Final

Ao final do projeto, o ambiente contava com uma VPC totalmente funcional, com:

- ✅ Sub-rede pública e sub-rede privada configuradas corretamente, cada uma com sua própria tabela de rota;
- ✅ Servidor bastion acessível pela internet, servindo como único ponto de entrada administrativo;
- ✅ Instância privada sem exposição direta à internet, mas com saída funcional através do NAT Gateway;
- ✅ Conectividade validada na prática, com login em cadeia (bastion → instância privada) e teste de acesso externo via `ping`.

<!--
  📌 ESPAÇO RESERVADO: PRINTS DO PAINEL AWS
  Inclua aqui capturas de tela do console AWS mostrando:
  - A VPC e as sub-redes criadas
  - As tabelas de rota pública e privada, com suas respectivas rotas
  - O servidor bastion e a instância privada em execução
  - O resultado do comando ping confirmando a saída via NAT Gateway
-->

---

## 8. Competências Desenvolvidas

<p align="center">
  <img src="https://img.shields.io/badge/Cloud%20Computing-232F3E?style=flat-square&logo=amazonaws&logoColor=white" alt="Cloud Computing">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=flat-square&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=flat-square&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Networking-8C4FFF?style=flat-square&logo=amazonaws&logoColor=white" alt="Networking">
</p>

- ☁️ Cloud Computing (AWS)
- 🌐 Amazon VPC (sub-redes, tabelas de rota, gateways)
- 🔐 Padrão de arquitetura Bastion Host
- 🔁 NAT Gateway para conectividade de saída segura
- 🛠️ Troubleshooting e validação de conectividade de rede
- 📝 Raciocínio arquitetural aplicado a redes em nuvem

---

<p align="center">
  <sub>Projeto desenvolvido como parte do <strong>Programa Re/Start AWS IA + No Code</strong>, Escola da Nuvem</sub>
</p>
