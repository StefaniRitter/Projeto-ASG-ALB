# Projeto sobre Auto Scaling & Load Balancer

Este projeto visa explicar, de forma resumida, o que são o Auto Scaling e o Load Balancer, e como esses recursos funcionam dentro da plataforma Amazon Web Services.

## ⚖️ Load Balancer

### O que é?
Um Load Balancer é um serviço que distribui o tráfego de forma inteligente entre várias instâncias/servidores. Ele funciona como uma porta de entrada única para os usuários.

### Como funciona?
1. Usuário acessa o site (requisição através de um único endereço DNS).
2. O Load Balancer:
   * Recebe e verifica quais instâncias estão disponíveis
   * Decide qual instância vai atender cada requisição, usando estratégias para distribuição de tráfego.
   * Monitora a saúde das instâncias (health checks).
   * Se uma instância falhar → ele remove do tráfego até ela ficar saudável.

### Configuração:
**1. Criação do grupo de destino**: O Load Balancer precisa saber para onde enviar o tráfego, e isso só é possível se associar um Grupo de Destino (Target Group), que é uma lista de recursos (targets) que podem receber tráfego do Load Balancer.

A criação é feita no Console da AWS, na seção EC2 em 'Grupos de Destino' -> 'Criar grupo de destino', onde são definidos:
 * Tipo de destino (EC2, IP, Lambda);
 * Nome;
 * Protocolo/Porta: HTTP na porta 80, por exemplo;
 * VPC;
 * Verificações de integridade;

Quando usado em conjunto com Auto Scaling, não é necessário adicionar instâncias, pois o Auto Scaling faz isso automaticamente.

**2. Criação do Load Balancer**:
Essa etapa acontece no Console da AWS, em EC2 -> Load Balancers -> Criar Load Balancer, onde é definido:
 * Tipo do balanceador de carga (para o resumo/projeto vamos considerar o uso do ALB):
      * Application Load Balancer (ALB): ideal para HTTP/HTTPS, entende camadas 7 (pode rotear por URL, headers).
      * Network Load Balancer (NLB): ideal para tráfego de rede TCP/UDP de alta performance.
      * Classic Load Balancer (CLB): mais antigo, usado para compatibilidade.
 * Nome;
 * Esquema usado (no caso do ALB):
   * Internet-facing:	Sites, APIs, apps que precisam ser acessados publicamente.
   * Internal: Microserviços privados, aplicações internas de empresa, comunicação entre backends.
 * Mapeamento de Rede (VPC, Zonas de Disponibilidade e Sub-redes);
 * Grupos de Segurança (é importante ter um grupo próprio para o ALB);
 * Listeners e Roteamento: HTTP na porta 80, por exemplo;
 * Grupo de destino: Criado anteriormente.

## 🖥️ Auto Scaling

### O que é?
Auto Scaling é um recurso que permite que a infraestrutura aumente ou diminua recursos automaticamente, de acordo com a demanda.

**Se trata de um mecanismo automatico que:**
* Monitora métricas (CPU, memória, requisições, filas);
* Cria novas instâncias (scale up) quando a carga sobe;
* Remove instâncias (scale down) quando a carga cai;
* Mantém sempre a quantidade certa de recursos, sem precisar de intervenção manual.

**Imaginando um site de E-commerce, por exemplo, o Auto Scaling funcionaria assim:**

* Durante a madrugada, quando há pouco tráfego → o Auto Scaling reduz para 2 instâncias.
* Pela manhã, mais pessoas acessam → sobe para 5 instâncias.
* Na Black Friday, o tráfego explode → escala para 20 instâncias.
* Depois do pico, vai reduzindo para o normal → economiza custos.

### Configuração
**1. Criação do modelo de execução**: Para o Auto Scaling funcionar, ele precisa de um modelo de execução, garantindo que todas as intâncias criadas sejam identicas. Por isso, antes de criar o Auto Scaling, deve-se ter um modelo de execução (launch template), no qual é definido:
  * AMI (Amazon Machine Image): o sistema operacional e aplicativos base.
  * Tipo de instância: t3.micro, t3.medium, etc.
  * Chave SSH: para acessar a instância se precisar.
  * Security groups: regras de firewall.
  * User Data Script: comandos automáticos para rodar ao iniciar, se necessário.
  * IAM Role: permissões.
  * Outras configurações como EBS volumes, tags, etc.

**2. Criação do Auto Scaling Group**:
  * Essa etapa acontece no Console da AWS, em EC2 -> Grupos Auto Scaling -> Criar Grupo de Auto Scaling, onde é definido:
    * Nome do grupo;
    * Modelo de execução (modelo das instâncias que serão criadas, como dito anteriormente);
    * Configurações de Rede (VPC, Sub-redes);
    * Balanceador de Carga (Grupo de destino);
    * Tamanho do grupo (capacidade desejada, capacidade mínima e máxima);
    * Políticas de Escalabilidade (Scaling Up e Scaling Down);
   
### Scaling Up/Out
Significa adicionar mais recursos quando a carga aumenta, garantindo que o sistema continue rápido mesmo com mais tráfego.

* **Pode ser feito de duas formas:**
  * **Vertical (Scaling Up)**: aumenta o poder de uma máquina (mais CPU, RAM, etc).
  * **Horizontal (Scaling Out)**: adiciona mais instâncias/servidores para dividir a carga.

* **Exemplo de regra:**
“Se a utilização média de CPU das instâncias for maior que 70% por 5 minutos, adicione mais 1 instância.”

### Scaling Down/In
Significa reduzir recursos quando a carga diminui, para evitar pagar por recursos que não estão sendo usados, economizando custos.

* **Também pode ser feito de duas formas:**
  * **Vertical (Scaling Down)**: reduzir o tamanho da máquina.
  * **Horizontal (Scaling In)**: remover instâncias/servidores ociosos.

* **Exemplo de regra:**
“Se a utilização média de CPU for menor que 30% por 10 minutos, remova 1 instância.”

## 🔑 Por que usar Load Balancer e Auto Scaling juntos

Quando se usa Auto Scaling:
* O número de instâncias muda automaticamente.
* O Load Balancer garante que os usuários sempre sejam redirecionados apenas para as instâncias ativas e saudáveis.

Ou seja:

* Auto Scaling escalou para cima -> Load Balancer começa a mandar tráfego para as novas instâncias.
* Auto Scaling escalou para baixo -> Load Balancer para de mandar tráfego para as instâncias que vão ser desligadas.
* Usuários nunca percebem quando uma instância entra ou sai

## ⚠️ Pontos importantes:

* Se alguma coisa for mudada na instância (como versão da aplicação, por exemplo), se cria uma nova versão do Launch Template para que as próximas instâncias usem a configuração atualizada.

* Sem Launch Template, o Auto Scaling não sabe como criar as instâncias.

* Grupos de segurança:
  * O grupo de segurança do (Application) Load Balancer deve permitir tráfego externo para o ALB, ou seja, regra de entrada HTTP (porta 80) com origem = 0.0.0.0/0 (qualquer IP);
  * O grupo de segurança das instâncias EC2 deve permitir tráfego somente do Load Balancer para as instâncias, ou seja, regras de entrada HTTP (porta 80) com origem = grupo de segurança do Load Balancer e opcionalmente SSH (porta 22) com origem = IP fixo (para administração);


 ## 💡 Principais Aprendizados

O projeto demonstrou que o uso combinado de Load Balancer e Auto Scaling é fundamental para construir aplicações modernas na nuvem, resultando em uma infraestrutura eficiente, confiável e econômica.

Com o Load Balancer, o tráfego é distribuído de forma inteligente entre as instâncias, garantindo alta disponibilidade e melhor experiência para os usuários, mesmo que uma instância apresente falhas.

Já o Auto Scaling permite que a quantidade de recursos seja ajustada automaticamente conforme a demanda, evitando desperdício de dinheiro em horários de baixo uso e garantindo performance durante picos de acesso.

Em conjunto, esses recursos proporcionam:

* Alta disponibilidade: a aplicação continua acessível mesmo em caso de falhas.
* Escalabilidade automática: aumenta ou reduz capacidade sem intervenção manual.
* Otimização de custos: paga-se apenas pelos recursos realmente necessários.
* Desempenho consistente: mantém a aplicação rápida mesmo com aumento de tráfego.
* Gerenciamento simplificado: menos esforço operacional, já que a AWS faz o trabalho pesado.

Assim, utilizar Load Balancer e Auto Scaling é uma prática essencial para aplicações modernas que precisam ser escaláveis, resilientes e econômicas na nuvem.
 





