# Elastic Compute Cloud (EC2)

Serviço de máquinas virtuais na nuvem.

---

## 1. Mecanismos de Armazenamento

**Elastic Block Storage (EBS)**
Para dados persistentes que independem do estado da instância.

---

**Instance Store**
Para dados temporários e de acesso rápido.

---

**Elastic File System (EFS)**
Para dados compartilhados entre várias instâncias.

---

**Simpes Storage Service (S3)**
Para dados de backup.

---

## 2. Rede e Segurança

**Elastic Fabric Adapter (EFA)**
Oferece escalabilidade, flexibilidade e elasticidade para aplicativos HPC.

Latência menor e mais consistente com maior throughput que canais tradicionais de TCP.

---

**Elastic IP**
IP público constante para a EC2.
Há um limite de 5 endereços Elastic IP por Region (por conta da escassez do IPv4).

É cobrada uma taxa a cada hora em que o Elastic IP não está sendo utilizado (taxa de conscientização). Use apenas em dev, em prod use Route 53/ Load Balancar para expor sua aplicação.

---

**Security Group**

Firewall da instância EC2.

Todo `inboud traffic` bloqueado por padrão.

Todo `outbound traffic` liberado por padrão.

Pode usar o mesmo SG para várias instâncias e é possível modificar as regras de um SG a qualquer momento.

Pode-se referenciar um Security Group em outro Security Group.

Security Group é `statefull` -> a resposta de um tráfego de entrada tem permissão para sair da instância mesmo que não tenha um outbound rule permitindo a saída do tráfego.

Exemplo: Se eu mandar uma requisição de ping para uma instância, ela me responde mesmo que não tenha uma outbound rule liberando a porta de ICMP para essa instância. Para impedir isso, é preciso configurar uma Network ACL para a subnet. NACL é `stateless` e, portanto, não permite automaticamente o tráfego de resposta.

Exemplo de regras de entrada para servidor web:

![](./imagens/sg_inbound.png)

---

Exemplo de regras de entrada para bancos de dados (a origem pode ser um CIDR ou ID de Security Group)

![](./imagens/sg_inbound_2.png)

---

Exemplo de regras de entrada para conexão remota a partir de outro computador

![](./imagens/sg_inbound_3.png)

---

Exemplo de regra de entrada para conectar em uma instância a partir de outra instância com o mesmo Security Group.

![](./imagens/sg_inbound_4.png)

---

## 3. User Data

Comando que é executado uma única vez quando sua instância é iniciada. O código roda com permissão de root user. Você pode rodar para fazer instalações e configurações iniciais.

---

## 4. Amazon Machine Image (AMI)

Imagem da máquina virtual.

Podem ser personalizadas usando `user data`. Também há uma variedade de AMIs disponíveis no AWS Marketplace (Alteryx, Tableau,...).

Escopo = VPC

Ciclo de vida da AMI

![](./imagens/ami.png)

Você pode criar novas instâncias a partir da AMI.

Você pode copiar uma AMI dentro da mesma Region ou em uma Region diferente.

Após criar uma AMI, você pode mantê-la privada e restrita somente para você ou compartilhá-la com uma lista específica de contas AWS.

Você também pode torná-la pública para toda a comunidade AWS.

---

## 5. Estados de uma Instância

![](./imagens/states.png)

**Hibernate**
Salva uma cópia da memória RAM no disco EBS da instância. Quando ela é reinicializada, ela volta no estado em que estava, com os mesmos dados na memória. É ideal para aplicações que demoram muito para carregar.

Você não é cobrado pela instância enquanto ela está no estado de `stopped`, mas é cobrado enquanto ela está em `stopping` pois neste instante os dados da RAM estão sendo transferidos para o disco EBS (você não é cobrado pela transferência, só pelo armazenamento).

Para poder usar o Hibernate, você deve habilitar essa opção no momento de criação da instância (não é possível habilitar em uma instância existente). Além disso, a criptografia do volume EBS deve estar habilitada.

Hibernate não é habilitado com Instance Store (por conta de não persistir dados nesse modelo de storage).

A RAM precisa ter menos de 150 GB e deve haver espaço suficiente no EBS para armazenar o conteúdo da RAM.

---

## 6. Definição de preço

**On demand**

Modalidade de gasto mais cara. Você paga por hora ou segundo de máquina em execução (com mínimo de 60 segundos).

Itens que compõem o custo da instância:

![](./imagens/price.png)

A cobrança em segundos é feita em cima de frações de horas de instância em execução.

Sua fatura também é composta por uma parcela associada a transferência de dados.

![](./imagens/price_2.png)

Muito indicada para instâncias de desenvolvimento, para workloads com curto ciclo de vida ou ciclo de vida desconhecido.

---

**Instâncias Reservadas (RI)**

Desconto de até 72% comparado com o on demand:

* RI Padrão: ideal para uso constante (até 72% desconcto);
* RI Conversível: capacidade de alterar os atributos da RI (54% de desconto) para valores iguais ou superiores;
* RI Programada: são disponibilizadas para lançamento dentro da janela de tempo reservada.

Você tem três opções de pagamento: adiantado integral, adiantado parcial e sem pagamento adiantado.

---

## 7. Hosts dedicados

Servidor físico totalmente dedicado ao seu uso. Você consegue especificar em qual host dedicado sua instância será executada após ela ter sido interrompida ou reiniciada. Você também pode permitir que o Amazon EC2 insira as instâncias automaticamente.

---

## 8. Spot Instances

Uso da capacidade ociosa da Amazon. Desconto de até 90% com relação ao On Demand.

Sinal de `rebalanceamento de instância spot` = sinal que avisa quando uma instância spot tem risco elevado de interrupção. O sinal oferece oportunidade de gerenciar proativamente a instância antes do aviso de interrupção de dois minutos.

---

## 9. Placement Group

Serve para determinar o posicionamento físico de um grupo de instâncias interdependentes. Pode ser:

* Cluster: agrupa instâncias em uma AZ. Entrega alta performance de rede, baixa latência para comunicação de nó a nó, típica de HPC;
* Partição: distribui instâncias entre partições lógicas, de modo que instâncias em uma partição não compartilhem hardware subjacente com grupos de instâncias em diferentes partições. Ideal para workloads distribuídos e replicados, como Hadoop e Kafka;
* Distribuição: posiciona estritamente um pequeno grupo de instâncias por hardware subjacente distinto a fim de reduzir falhas correlacionadas. As instâncias são colocadas em racks diferentes.

Não há custo para criação de um placement group.

---

## 10. Rede

**Interface de Rede**
É como se fosse a placa de rede da instância. Instâncias com várias placas de rede oferecem maior performance de rede, incluindo recursos de largura de banda acima de 100 Gbps e maior performance de taxa de pacotes.

---

**EBS**
Quando o volume EBS raiz de uma instâncias é substituído, a instância é reinicializada e o conteúdo da memória RAM se perde.

Não é possível substituir o volume raiz do Instance Store.

---

## 11. Problemas de execução de instâncias

![](./imagens/problem.png)

---

**Problemas de timeout na conexão**

* Cheque o Security Group da instância e veja se o acesso de entrada é permitido na porta desejada;
* Cheque o Route Table da VPC. Você precisa de uma rota que envie todo o tráfego que sai da VPC para o Internet Gateway;
* Verifique o Network ACL da subnet. Ele deve permitir o tráfego de entrada do seu IP na porta desejada e também o tráfego de saída para as portas efêmeras `(1024-65535)`;
* Cheque o firewall do seu computador, se é permitido acesso a porta desejada;
* Verifique se sua instância EC2 tem um endereço IPv4 público;
* Verifique a carga de CPU da sua instância.