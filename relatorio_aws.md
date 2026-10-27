# Relatório Prático: Configuração de Ambiente AWS (VPC e EC2)

Neste relatório, apresento o passo a passo da criação manual de um ambiente de testes na AWS. O objetivo desta atividade prática foi configurar a infraestrutura de rede necessária — desde a fundação da VPC até as regras de firewall — para hospedar e acessar remotamente uma instância EC2 de forma segura, compreendendo na prática como os recursos se conectam.

## 1. VPC (Virtual Private Cloud)

Para isolar nosso ambiente de laboratório, criei primeiro a VPC (Virtual Private Cloud). Ela funciona como a nossa rede de fundação na AWS, operando como um datacenter virtual logicamente isolado. Configurei a VPC com o bloco CIDR `10.0.0.0/16`, o que me garante um amplo espaço de endereçamento IP privado para alocar todos os recursos de forma segura e organizada, estabelecendo a base para o restante da arquitetura.

![Acessando o serviço VPC](imagens/01-vpc-servico.png)
*Nota: Acessando o painel principal e selecionando o serviço VPC.*

![Painel inicial da VPC](imagens/02-vpc-painel.png)
*Nota: Painel inicial da VPC, com a visão geral dos recursos da região.*

![Criando a VPC](imagens/03-vpc-criacao.png)
*Nota: Iniciando a criação da VPC, optando por "Somente VPC" e definindo o bloco CIDR 10.0.0.0/16.*

![Confirmando a criação](imagens/04-vpc-confirmacao.png)
*Nota: Confirmando as configurações e o nome (vpc-lab) antes de finalizar a criação.*

## 2. Sub-rede Pública

Em seguida, criei uma Sub-rede pública dentro dessa VPC, usando o bloco menor `10.0.1.0/24`. Para garantir que as instâncias lançadas nesta sub-rede estivessem acessíveis externamente, ativei a opção de atribuição automática de IP público (Auto-assign public IPv4 address). Isso é fundamental, pois o objetivo da atividade é que a instância EC2 que vai rodar aqui dentro possa se comunicar e ser acessada diretamente da internet.

![Listagem de sub-redes](imagens/05-subnet-lista.png)
*Nota: Acessando a listagem de sub-redes.*

![Selecionando a VPC na sub-rede](imagens/06-subnet-vpc.png)
*Nota: Selecionando a vpc-lab recém-criada para atrelá-la à nova sub-rede.*

![Configurando a sub-rede](imagens/07-subnet-configuracao.png)
*Nota: Definindo o nome "subnet-lab" e o bloco CIDR 10.0.1.0/24.*

![Sub-rede criada](imagens/08-subnet-sucesso.png)
*Nota: Confirmação de que a sub-rede foi criada com sucesso.*

![Editando configurações de IP](imagens/09-subnet-editar-ip.png)
*Nota: Acessando as configurações da sub-rede para habilitar o IP público.*

![Marcando auto-assign IP](imagens/10-subnet-marcar-ip.png)
*Nota: Marcando a opção "Habilitar endereço IPv4 público de atribuição automática".*

![Confirmação de IP automático](imagens/11-subnet-ip-sucesso.png)
*Nota: Mensagem confirmando a alteração nas configurações da sub-rede.*

## 3. Internet Gateway (IGW)

Para que a rede realmente tenha comunicação bidirecional com o mundo externo, instanciei um Internet Gateway (IGW). Após a criação, fiz o attach (anexei) deste recurso à minha VPC. O IGW atua como a porta de saída e entrada da nossa rede virtual para a internet, sendo um componente altamente disponível e escalável fornecido pela AWS que permite a comunicação da nossa VPC com a web.

![Painel IGW](imagens/12-igw-painel.png)
*Nota: Painel de Internet Gateways, preparando para criar um novo recurso.*

![Criando IGW](imagens/13-igw-criar.png)
*Nota: Definindo o nome igw-lab para o Gateway.*

![IGW Detached](imagens/14-igw-detached.png)
*Nota: Internet Gateway criado, porém no estado Detached (Desanexado).*

![Associando à VPC](imagens/15-igw-attach.png)
*Nota: Realizando a associação (Attach) do Gateway à nossa vpc-lab.*

![IGW Attached](imagens/16-igw-attached.png)
*Nota: Confirmação do estado Attached, garantindo acesso à internet.*

## 4. Tabela de Rotas (Route Table)

Como apenas anexar o IGW não é suficiente para rotear o tráfego, precisei configurar uma Tabela de Rotas customizada. Criei a tabela e adicionei explicitamente uma rota (destino `0.0.0.0/0`) apontando para o nosso Internet Gateway recém-criado. Por fim, associei essa tabela de rotas especificamente à nossa sub-rede. É essa associação de uma rota externa para um IGW que tecnicamente transforma uma sub-rede privada em uma sub-rede pública.

![Painel Tabela de Rotas](imagens/17-rt-painel.png)
*Nota: Acessando a seção de Tabelas de rotas.*

![Criando Tabela de Rotas](imagens/18-rt-criar.png)
*Nota: Criando a tabela table-lab e vinculando à vpc-lab.*

![Tabela Criada](imagens/19-rt-sucesso.png)
*Nota: Tabela criada com sucesso.*

![Editando Rotas](imagens/20-rt-editar.png)
*Nota: Editando as rotas da tabela.*

![Adicionando Rota Externa](imagens/21-rt-add-rota.png)
*Nota: Adicionando a rota 0.0.0.0/0 apontando para o nosso Internet Gateway.*

![Rotas atualizadas](imagens/22-rt-rota-sucesso.png)
*Nota: Confirmação das rotas atualizadas.*

![Aba associações](imagens/23-rt-associacoes.png)
*Nota: Visualizando as associações de sub-rede atuais.*

![Editando associações](imagens/24-rt-editar-associacao.png)
*Nota: Editando as associações para vincular a subnet-lab à tabela.*

![Associação com sucesso](imagens/25-rt-associacao-sucesso.png)
*Nota: Associação da sub-rede concluída com sucesso.*

## 5. Security Group

Como medida de segurança e proteção no nível da interface de rede (firewall virtual da instância), configurei um Security Group. Adicionei uma regra de entrada liberando exclusivamente o tráfego do protocolo TCP na porta 22 (SSH). Para seguir as melhores práticas de privilégio mínimo, restringi a Origem (Source) estritamente para o meu próprio IP público (My IP), garantindo que apenas a minha conexão atual tenha permissão para acessar o servidor e bloqueando o restante da internet.

![Painel Security Group](imagens/26-sg-painel.png)
*Nota: Painel de Security Groups, iniciando a criação do grupo.*

![Regras Security Group](imagens/27-sg-regras.png)
*Nota: Configurando a regra de entrada SSH (porta 22) com origem restrita ao "Meu IP".*

![Security Group criado](imagens/28-sg-sucesso.png)
*Nota: Security Group criado e com as regras validadas.*

## 6. Instância EC2 (Máquina Virtual)

Com toda a infraestrutura de rede e segurança devidamente provisionada, fiz o deploy da máquina virtual (Instância EC2). Escolhi a imagem do Amazon Linux 2023 por sua leveza e otimização para a nuvem AWS. Selecionei o tipo de instância `t3.micro`, vinculei explicitamente a máquina à nossa VPC e à Sub-rede pública, anexei o Security Group criado no passo anterior e gerei um novo par de chaves assimétricas (`.pem`) para garantir uma autenticação segura e criptografada.

![Serviço EC2](imagens/29-ec2-servico.png)
*Nota: Acessando o serviço EC2 no console.*

![Painel de Instâncias](imagens/30-ec2-painel.png)
*Nota: Painel principal do EC2 para iniciar uma nova instância.*

![Nome da Instância](imagens/31-ec2-nome.png)
*Nota: Definindo o nome da instância (capacita-lab).*

![Escolhendo a AMI](imagens/32-ec2-ami.png)
*Nota: Selecionando a imagem do Amazon Linux 2023.*

![Criando Par de Chaves](imagens/33-ec2-chaves.png)
*Nota: Criando e baixando um novo par de chaves (.pem) para autenticação segura.*

![Configuração de Rede EC2](imagens/34-ec2-rede.png)
*Nota: Editando a rede para vincular nossa vpc-lab, subnet-lab e o grupo de segurança.*

![Revisão EC2](imagens/35-ec2-revisao.png)
*Nota: Revisão final das configurações antes do lançamento.*

![Lançamento EC2](imagens/36-ec2-lancamento.png)
*Nota: Sucesso ao iniciar a execução da nova instância.*

![Instância Executando](imagens/37-ec2-executando.png)
*Nota: Instância EC2 provisionada, listada no estado Executando.*

## 7. Acesso via SSH (Terminal)

Por fim, realizei o teste prático de conectividade para validar a arquitetura. Pelo terminal do meu sistema (baseado em Linux), ajustei as permissões de leitura do arquivo da chave privada com o comando `chmod 400` para protegê-lo, exigência obrigatória do SSH. Em seguida, iniciei a conexão utilizando o usuário padrão `ec2-user`, a chave `.pem` e o IP público da instância. O acesso foi estabelecido com sucesso, comprovando o correto funcionamento da rede e do firewall.

![Terminal SSH](imagens/38-ssh-terminal.png)
*Nota: Terminal mostrando o acesso bem-sucedido via SSH, utilizando a chave privada e o IP público da instância.*