# Good Video
Documentação geral do projeto goodvideo.   
Trabalho final da Pós Tech FIAP (2025).  
[Descrição do desafio.](doc/Hack%20SOAT%206_7.pdf)

## Repositórios
- [Serviço Usuário](https://github.com/goodvideo-postech-org/goodvideo-auth)
- [Serviço Upload](https://github.com/goodvideo-postech-org/goodvideo-upload)
- [Serviço Processamento](https://github.com/goodvideo-postech-org/goodvideo-video-processor)

## Sonar
[Sonar](https://sonarcloud.io/organizations/goodvideo-postech-org/projects?sort=name)

## Event storming / DDD
[Link do board no Miro](https://miro.com/app/board/uXjVLxYrinA=/?share_link_id=903786219199)

## Decisões de negócio
- O usuário precisará de cadastro prévio para poder utilizar a plataforma;
- A senha do usuário deverá ser criptografada no banco;
- O arquivo deverá ser no formato mp4/video;
- O vídeo será limitado no tamanho de 10Mb;
- Cada usuário deverá possuir uma pasta com seus respectivos processamentos salvos também em pastas;
- O arquivo de vídeo deverá ser excluído após o processamento;
- O arquivo ZIP terá validade de 24h após o processamento;
- Caso ocorra algum erro durante o processamento, o usuário será notificado e deverá reenviar o vídeo para nova tentativa;
- Caso ocorra algum erro, tanto o vídeo quanto o ZIP devem ser apagados.

## Arquitetura Geral
- **Plataforma:** 
  - O sistema utiliza a AWS como base, aproveitando ferramentas integradas e opções gratuitas (free tier) para prototipação e desenvolvimento. A escolha foi baseada na familiaridade do time e no suporte robusto da plataforma.
- **Contextos Delimitados:** 
  - A aplicação foi dividida em dois domínios principais para promover modularidade e organização:
    - **Usuário:** Responsável pelo gerenciamento de usuários e autenticação.
    - **Processamento:** Cuida do upload, processamento e armazenamento de vídeos.

### Contexto Usuário
- **Funcionalidades:**
  - **Cadastro de Usuários:** Apenas dados básicos (nome, e-mail, etc.) são armazenados, evitando informações sensíveis para reduzir riscos e atender à LGPD.
  - **Autenticação:** Implementada com **JWT (JSON Web Tokens)**, garantindo autenticação e autorização seguras e eficientes.
- **Infraestrutura:**
  - O serviço foi projetado para **escalabilidade horizontal**, utilizando um **load balancer** para distribuir requisições HTTP.
  - O banco de dados utilizado é o **PostgreSQL**, configurado para suportar **escalabilidade vertical**, uma solução suficiente para a natureza do crescimento controlado dos dados de usuários.

### Contexto Processamento
- **Serviço de Upload:**
  - Permite que usuários enviem vídeos, com validação de formato e tamanho. Os arquivos são armazenados no **AWS S3**, aproveitando baixo custo, alta durabilidade e integração com outros serviços da AWS.
  - Atualiza o status do processamento dos vídeos ao receber notificações do serviço de processamento.
  - Utiliza o **Kafka** como broker de mensageria, garantindo comunicação assíncrona e desacoplada entre os serviços.
- **Serviço Video-Processor:**
  - Realiza o processamento de vídeos consumindo mensagens de uma fila Kafka.
  - Os resultados são armazenados no AWS S3, e o serviço opera de forma assíncrona, sem expor endpoints HTTP.
- **Mensageria (Kafka):**
  - **Fila de Upload:** Gerencia tarefas de processamento para cada vídeo enviado.
  - **Fila de Notificação:** Notifica quando o processamento é concluído, permitindo que o status do vídeo seja atualizado e o arquivo final seja disponibilizado para download.
  - O Kafka garante escalabilidade e alta disponibilidade, além de permitir o processamento distribuído de mensagens.

### Segurança e Conformidade
- **Proteções Implementadas:**
  - Todas as APIs são protegidas com autenticação JWT, garantindo que apenas usuários autenticados tenham acesso ao sistema.
  - Dados em trânsito são protegidos por **HTTPS**, e dados armazenados em repouso (como arquivos temporários) podem ser criptografados com ferramentas da AWS.
  - O broker Kafka é acessível apenas pela rede interna da cloud, aumentando a segurança contra acessos externos.
  - Os serviços da núvem AWS são configurados dentro de uma **VPC (Virtual Private Cloud)** para isolar a rede e proteger o acesso. A VPC garante que o banco só possa ser acessado pelos serviços autorizados dentro do ambiente de produção.
- **Conformidade com LGPD:**
  - O sistema armazena informações mínimas e evita dados sensíveis de usuários.
  - Vídeos são excluídos automaticamente após o processamento, e arquivos ZIP são armazenados por, no máximo, 24 horas.
  - A segregação de diretórios por usuário garante isolamento e privacidade nos dados armazenados.

### Práticas de Desenvolvimento
- **Clean Architecture:**
  - O sistema segue os princípios de Clean Architecture para garantir modularidade, separação de responsabilidades e fácil manutenção.
- **Testes Automatizados:**
  - Testes unitários verificam funcionalidades isoladas, enquanto testes baseados em **BDD (Behavior-Driven Development)** validam o comportamento do sistema com base em requisitos de negócios.
- **Pipeline CI/CD:**
  - O pipeline de integração e entrega contínuas automatiza builds, testes (com cobertura de código medida via **Sonar**) e deploy.
  - Políticas de PR e proteção de branch garantem qualidade no código antes da integração.
- **Orquestração com Kubernetes:**
  - Todos os serviços são orquestrados no **Kubernetes**, permitindo escalabilidade, gerenciamento eficiente e alta confiabilidade em ambientes de produção.

## Diagramas
Para editar, abrir os diagramas no [App Diagrams.net](https://app.diagrams.net/)

### Diagrama de Arquitetura
<img src="diagramas/diagrama%20de%20arquitetura.drawio.png" width="250">

### Diagrama de Sequência
<img src="diagramas/fluxograma.drawio.png" width="250">

## Tecnologias
- Java 17;
- Postgres;
- AWS S3 / AWS RDS;
- Kafka;
- JWT;
- Kubernetes;

# Equipe
Carlos Bridi - RM355971  
Nicollas P. Eissmann - RM355576  
Daniel R. Martini - RM355054  
Roberto Debarba - RM355038
