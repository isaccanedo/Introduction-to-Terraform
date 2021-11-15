# 1. Visão Geral
Infraestrutura como código (IaC) é uma prática que se tornou popular com a popularidade crescente de provedores de nuvem pública, como AWS, Google e Microsoft. Em suma, consiste em gerenciar um conjunto de recursos (computação, rede, armazenamento, etc.) usando a mesma abordagem que os desenvolvedores usam para gerenciar o código do aplicativo.

Neste tutorial, faremos um rápido tour pelo Terraform, uma das ferramentas mais populares usadas pelas equipes de DevOps para automatizar tarefas de infraestrutura. O principal apelo do Terraform é que apenas declaramos como nossa infraestrutura deve ser, e a ferramenta decidirá quais ações devem ser tomadas para “materializar” essa infraestrutura.

# 2. Uma breve história
De acordo com o GitHub, a primeira data de confirmação do Terraform foi em 21 de maio de 2014. O autor foi Mitchell Hashimoto, um dos fundadores da Hashicorp, e contém apenas um arquivo README que descreve o que podemos chamar de “declaração de missão”:

```
Terraform é uma ferramenta para construir e mudar a infraestrutura de forma segura [sic] e eficiente.
```

Esta frase descreve muito bem sua intenção. Desde então, a ferramenta tem aumentado constantemente seus recursos em termos de provedores de infraestrutura com suporte.

No momento em que este livro foi escrito, o Terraform oferece suporte oficialmente a cerca de 130 provedores. A página de seus provedores com suporte da comunidade lista outros 160. Alguns desses provedores expõem apenas alguns recursos, mas outros, como AWS ou Azure, têm centenas deles.


freestar
Esse grande número de recursos com suporte torna o Terraform uma ferramenta de escolha para muitos engenheiros de DevOps. Além disso, usar uma única ferramenta para gerenciar vários fornecedores é uma grande vantagem.

# 3. Olá, Terraform
Antes de entrar em mais detalhes de seu funcionamento interno, vamos começar com as coisas básicas: configuração inicial e um projeto rápido no estilo “Hello, World”.

### 3.1. Baixar e instalar
A distribuição do Terraform consiste em um único arquivo binário que podemos baixar gratuitamente na página de download da Hashicorp. Não há dependências e podemos simplesmente executá-lo copiando o binário executável para alguma pasta no PATH de nosso sistema operacional.

Depois de concluir esta etapa, podemos verificar se ele está funcionando corretamente com um comando simples:

```
$ terraform -v
Terraform v0.12.24
```

É isso - sem privilégios de administrador necessários! Podemos obter ajuda rápida dos comandos disponíveis executando o Terraform sem nenhum argumento:

```
$ terraform
Usage: terraform [-version] [-help] <command> [args]
... help content omitted
```

### 3.2. Criando nosso primeiro projeto
Um projeto Terraform é apenas um conjunto de arquivos em um diretório contendo definições de recursos. Esses arquivos, que por convenção terminam em .tf, usam a linguagem de configuração do Terraform para definir os recursos que queremos criar.

Para o nosso projeto “Hello, Terraform”, nosso recurso será apenas um arquivo com conteúdo fixo. Vamos ver como fica abrindo o shell de comando e digitando alguns comandos:

```
$ cd $HOME
$ mkdir hello-terraform
$ cd hello-terraform
$ cat > main.tf <<EOF
provider "local" {
  version = "~> 1.4"
}
resource "local_file" "hello" {
  content = "Hello, Terraform"
  filename = "hello.txt"
}
EOF
```

O arquivo main.tf contém dois blocos: uma declaração do provedor e uma definição de recurso. A declaração do provedor afirma que usaremos o provedor local na versão 1.4 ou um compatível.

A seguir, temos uma definição de recurso chamada hello do tipo local_file. Este tipo de recurso, como o nome indica, é apenas um arquivo no sistema de arquivos local com o conteúdo fornecido.

### 3.3. init, planejar e aplicar
Agora, vamos prosseguir e executar o Terraform neste projeto. Como esta é a primeira vez que executamos este projeto, precisamos inicializá-lo com o comando init:

```
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "local" (hashicorp/local) 1.4.0...

Terraform has been successfully initialized!
... more messages omitted
```

Nesta etapa, o Terraform verifica nossos arquivos de projeto e baixa qualquer provedor necessário - o provedor local, em nosso caso.

Em seguida, usamos o comando plan para verificar quais ações o Terraform executará para criar nossos recursos. Esta etapa funciona quase como o recurso de "simulação" disponível em outros sistemas de compilação, como a ferramenta make do GNU:

```
$ terraform plan
... messages omitted
Terraform will perform the following actions:

  # local_file.hello will be created
  + resource "local_file" "hello" {
      + content              = "Hello, Terraform"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "hello.txt"
      + id                   = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
... messages omitted
```

Aqui, o Terraform está nos dizendo que precisa criar um novo recurso, o que é esperado porque ainda não existe. Também podemos ver os valores fornecidos que definimos e um par de atributos de permissão. Como não os fornecemos em nossa definição de recurso, o provedor assumirá os valores padrão.

Agora podemos prosseguir para a criação real de recursos usando o comando apply:

```
$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # local_file.hello will be created
  + resource "local_file" "hello" {
      + content              = "Hello, Terraform"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "hello.txt"
      + id                   = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

local_file.hello: Creating...
local_file.hello: Creation complete after 0s [id=392b5481eae4ab2178340f62b752297f72695d57]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

Agora podemos verificar se o arquivo foi criado com o conteúdo especificado:

```
$ cat hello.txt
Hello, Terraform
```

Tudo certo! Agora, vamos ver o que acontece se executarmos novamente o comando apply, desta vez usando a sinalização -auto-approve para que o Terraform vá imediatamente sem pedir qualquer confirmação:

```
$ terraform apply -auto-approve
local_file.hello: Refreshing state... [id=392b5481eae4ab2178340f62b752297f72695d57]

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
```

Desta vez, o Terraform não fez nada porque o arquivo já existia. Mas isso não é tudo. Às vezes, existe um recurso, mas alguém pode ter alterado um de seus atributos, um cenário geralmente conhecido como “desvio de configuração”. Vamos ver como o Terraform se comporta neste cenário:

```
$ echo foo > hello.txt
$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

local_file.hello: Refreshing state... [id=392b5481eae4ab2178340f62b752297f72695d57]

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # local_file.hello will be created
  + resource "local_file" "hello" {
      + content              = "Hello, Terraform"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "hello.txt"
      + id                   = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
... more messages omitted
```

Terraform detectou a mudança no conteúdo do arquivo hello.txt e gerou um plano para restaurá-lo. Como o provedor local não tem suporte para modificação no local, vemos que o plano consiste em uma única etapa - recriar o arquivo.

Agora podemos executar o apply novamente e, como resultado, ele restaurará o conteúdo do arquivo ao conteúdo pretendido:

```
$ terraform apply -auto-approve
... messages omitted
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

$ cat hello.txt
Hello, Terraform
```

# 4. Conceitos Básicos
Agora que cobrimos o básico, vamos entrar nos conceitos básicos do Terraform.

### 4.1. Provedores
Um provedor funciona quase como um driver de dispositivo do sistema operacional. Ele expõe um conjunto de tipos de recursos usando uma abstração comum, mascarando os detalhes de como criar, modificar e destruir um recurso de maneira bastante transparente para os usuários.

O Terraform baixa provedores automaticamente de seu registro público conforme necessário, com base nos recursos de um determinado projeto. Ele também pode usar plug-ins personalizados, que devem ser instalados manualmente pelo usuário. Finalmente, alguns provedores integrados fazem parte do binário principal e estão sempre disponíveis.

Com algumas exceções, usar um provedor requer configurá-lo com alguns parâmetros. Eles variam muito de provedor para provedor, mas, em geral, precisaremos fornecer credenciais para que ele possa acessar sua API e enviar solicitações.

Embora não seja estritamente necessário, é considerado uma boa prática declarar explicitamente qual provedor usaremos em nosso projeto Terraform e informar sua versão. Para isso, usamos o atributo version disponível para qualquer declaração de provedor:

```
provider "kubernetes" {
  version = "~> 1.10"
}
```

Aqui, como não estamos fornecendo parâmetros adicionais, o Terraform procurará em outro lugar os necessários. Nesse caso, a implementação do provedor procura parâmetros de conexão usando os mesmos locais usados por kubectl. Outros métodos comuns são o uso de variáveis de ambiente e arquivos de variáveis, que são apenas arquivos contendo pares de valores-chave.

### 4.2. Recursos
No Terraform, um recurso é qualquer coisa que pode ser um alvo para operações CRUD no contexto de um determinado provedor. Alguns exemplos são uma instância EC2, um Azure MariaDB ou uma entrada DNS.

Vejamos uma definição simples de recurso:

```
resource "aws_instance" "web" {
  ami = "some-ami-id"
  instance_type = "t2.micro"
}
```

Primeiro, sempre temos a palavra-chave resource que inicia uma definição. Em seguida, temos o tipo de recurso, que geralmente segue a convenção provider_type. No exemplo acima, aws_instance é um tipo de recurso definido pelo provedor AWS, usado para definir uma instância EC2. Depois disso, há o nome do recurso definido pelo usuário, que deve ser exclusivo para este tipo de recurso no mesmo módulo - mais sobre os módulos posteriormente.

Finalmente, temos um bloco contendo uma série de argumentos usados como especificação de recursos. Um ponto-chave sobre os recursos é que, uma vez criados, podemos usar expressões para consultar seus atributos. Além disso, e igualmente importante, podemos usar esses atributos como argumentos para outros recursos.

Para ilustrar como isso funciona, vamos expandir o exemplo anterior criando nossa instância EC2 em uma VPC (nuvem privada virtual) não padrão:

```
resource "aws_instance" "web" {
  ami = "some-ami-id"
  instance_type = "t2.micro"
  subnet_id = aws_subnet.frontend.id
}
resource "aws_subnet" "frontend" {
  vpc_id = aws_vpc.apps.id
  cidr_block = "10.0.1.0/24"
}
resource "aws_vpc" "apps" {
  cidr_block = "10.0.0.0/16"
}
```

Aqui, usamos o atributo id de nosso recurso VPC como o valor para o argumento vpc_id do front-end. Em seguida, seu parâmetro id se torna o argumento para a instância EC2. Observe que esta sintaxe específica requer Terraform versão 0.12 ou posterior. As versões anteriores usavam uma sintaxe “$ {expression}” mais complicada, que ainda está disponível, mas é considerada legada.

Este exemplo também mostra um dos pontos fortes do Terraform: independentemente da ordem em que declaramos os recursos em nosso projeto, ele descobrirá a ordem correta em que deve criá-los ou atualizá-los com base em um gráfico de dependência que constrói ao analisá-los.

### 4.3. count e for_each Meta Argumentos
Os meta-argumentos count e for_each nos permitem criar várias instâncias de qualquer recurso. A principal diferença entre eles é que count espera um número não negativo, enquanto for_each aceita uma lista ou mapa de valores.

Por exemplo, vamos usar a contagem para criar algumas instâncias EC2 na AWS:

```
resource "aws_instance" "server" {
  count = var.server_count 
  ami = "ami-xxxxxxx"
  instance_type = "t2.micro"
  tags = {
    Name = "WebServer - ${count.index}"
  }
}
```

Dentro de um recurso que usa count, podemos usar o objeto count em expressões. Este objeto possui apenas uma propriedade: índice, que contém o índice (baseado em zero) de cada instância.

Da mesma forma, podemos usar o meta-argumento for_each para criar essas instâncias com base em um mapa:

```
variable "instances" {
  type = map(string)
}
resource "aws_instance" "server" {
  for_each = var.instances 
  ami = each.value
  instance_type = "t2.micro"
  tags = {
    Name = each.key
  }
}
```

Desta vez, usamos um mapa de rótulos para nomes AMI (Amazon Machine Image) para criar nossos servidores. Dentro de nosso recurso, podemos usar cada objeto, o que nos dá acesso à chave e ao valor atuais para uma instância particular.

Um ponto chave sobre count e for_each é que, embora possamos atribuir expressões a eles, o Terraform deve ser capaz de resolver seus valores antes de realizar qualquer ação de recurso. Como resultado, não podemos usar uma expressão que dependa de atributos de saída de outros recursos.

### 4.4. Fontes de dados
As fontes de dados funcionam praticamente como recursos “somente leitura”, no sentido de que podemos obter informações sobre as existentes, mas não podemos criá-las ou alterá-las. Eles geralmente são usados para buscar parâmetros necessários para criar outros recursos.

Um exemplo típico é a fonte de dados aws_ami disponível no provedor AWS, que usamos para recuperar atributos de um AMI existente:

```
data "aws_ami" "ubuntu" {
  most_recent = true
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-trusty-14.04-amd64-server-*"]
  }
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  owners = ["099720109477"] # Canonical
}
```

Este exemplo define uma fonte de dados chamada “ubuntu” que consulta o registro AMI e retorna vários atributos relacionados à imagem localizada. Podemos então usar esses atributos em outras definições de recursos, acrescentando o prefixo de dados ao nome do atributo:

```
resource "aws_instance" "web" {
  ami = data.aws_ami.ubuntu.id 
  instance_type = "t2.micro"
}
```

### 4.5. Estado
O estado de um projeto Terraform é um arquivo que armazena todos os detalhes sobre os recursos que foram criados no contexto de um determinado projeto. Por exemplo, se declararmos um recurso azure_resourcegroup em nosso projeto e executar o Terraform, o arquivo de estado armazenará seu identificador.

O objetivo principal do arquivo de estado é fornecer informações sobre os recursos já existentes, então, quando modificarmos nossas definições de recursos, o Terraform poderá descobrir o que precisa fazer.

Um ponto importante sobre os arquivos de estado é que eles podem conter informações confidenciais. Os exemplos incluem senhas iniciais usadas para criar um banco de dados, chaves privadas e assim por diante.

O Terraform usa o conceito de back-end para armazenar e recuperar arquivos de estado. O back-end padrão é o back-end local, que usa um arquivo na pasta raiz do projeto como seu local de armazenamento. Também podemos configurar um back-end remoto alternativo, declarando-o em um bloco terraform em um dos arquivos .tf do projeto:

```
terraform {
  backend "s3" {
    bucket = "some-bucket"
    key = "some-storage-key"
    region = "us-east-1"
  }
}
```

### 4.6. Módulos
Os módulos Terraform são o principal recurso que nos permite reutilizar as definições de recursos em vários projetos ou simplesmente ter uma melhor organização em um único projeto. Isso é muito parecido com o que fazemos na programação padrão: em vez de um único arquivo contendo todo o código, organizamos nosso código em vários arquivos e pacotes.

Um módulo é apenas um diretório que contém um ou mais arquivos de definição de recursos. Na verdade, mesmo quando colocamos todo o nosso código em um único arquivo / diretório, ainda estamos usando módulos - neste caso, apenas um. O ponto importante é que os subdiretórios não são incluídos como parte de um módulo. Em vez disso, o módulo pai deve incluí-los explicitamente usando a declaração do módulo:

```
module "networking" {
  source = "./networking"
  create_public_ip = true
}
```

Aqui, estamos referenciando um módulo localizado no subdiretório “networking” e passando um único parâmetro para ele - um valor booleano neste caso.

É importante observar que em sua versão atual, o Terraform não permite o uso de count e for_each para criar várias instâncias de um módulo.

### 4.7. Variáveis de entrada
Qualquer módulo, incluindo o superior ou principal, pode definir várias variáveis de entrada usando definições de bloco de variável:

```
variable "myvar" {
  type = string
  default = "Some Value"
  description = "MyVar description"
}
```

Uma variável possui um tipo, que pode ser uma string, um mapa ou um conjunto, entre outros. Ele também pode ter um valor padrão e uma descrição. Para variáveis definidas no módulo de nível superior, o Terraform atribuirá valores reais a uma variável usando várias fontes:

- opção de linha de comando -var;
- arquivos .tfvar, usando opções de linha de comando ou verificando arquivos / locais conhecidos;
- Variáveis de ambiente começando com TF_VAR_;
- O valor padrão da variável, se houver.

Quanto às variáveis definidas em módulos aninhados ou externos, qualquer variável que não tenha valor padrão deve ser fornecida usando argumentos em uma referência de módulo. O Terraform gerará um erro se tentarmos usar um módulo que requer um valor para uma variável de entrada, mas não fornecermos um.

Depois de definidas, podemos usar variáveis em expressões usando o prefixo var:

```
resource "xxx_type" "some_name" {
  arg = var.myvar
}
```

### 4.8. Valores de saída
Por design, o consumidor de um módulo não tem acesso a nenhum recurso criado dentro do módulo. Às vezes, no entanto, precisamos de alguns desses atributos para usar como entrada para outro módulo ou recurso. Para resolver esses casos, um módulo pode definir blocos de saída que expõem um subconjunto dos recursos criados:

```
output "web_addr" {
  value = aws_instance.web.private_ip
  description = "Web server's private IP address"
}
```

Aqui, estamos definindo um valor de saída denominado “web_addr” contendo o endereço IP de uma instância EC2 que nosso módulo criou. Agora, qualquer módulo que faz referência ao nosso módulo pode usar esse valor em expressões como module.module_name.web_addr, onde module_name é o nome que usamos na declaração do módulo correspondente.

### 4.9. Variáveis Locais
Variáveis locais funcionam como variáveis padrão, mas seu escopo é limitado ao módulo onde são declaradas. O uso de variáveis locais tende a reduzir a repetição de código, especialmente ao lidar com valores de saída de módulos:

```
locals {
  vpc_id = module.network.vpc_id
}
module "network" {
  source = "./network"
}
module "service1" {
  source = "./service1"
  vpc_id = local.vpc_id
}
module "service2" {
  source = "./service2"
  vpc_id = local.vpc_id
}
```

Aqui, a variável local vpc_id recebe o valor de uma variável de saída do módulo de rede. Posteriormente, passamos esse valor como um argumento para os módulos serviço1 e serviço2.

### 4.10. Espaços de trabalho
Os espaços de trabalho do Terraform nos permitem manter vários arquivos de estado para o mesmo projeto. Quando executamos o Terraform pela primeira vez em um projeto, o arquivo de estado gerado irá para o espaço de trabalho padrão. Posteriormente, podemos criar um novo espaço de trabalho com o comando terraform workspace new, opcionalmente fornecendo um arquivo de estado existente como parâmetro.

Podemos usar espaços de trabalho da mesma forma que usaríamos ramos em um VCS normal. Por exemplo, podemos ter um espaço de trabalho para cada ambiente de destino - DEV, QA, PROD - e, ao alternar os espaços de trabalho, podemos terraformar e aplicar as alterações à medida que adicionamos novos recursos.

Dada a forma como isso funciona, os espaços de trabalho são uma excelente escolha para gerenciar várias versões - ou “encarnações”, se preferir - do mesmo conjunto de configurações. Esta é uma ótima notícia para todos que tiveram que lidar com o infame problema “funciona no meu ambiente”, pois nos permite garantir que todos os ambientes tenham a mesma aparência.

Em alguns cenários, pode ser conveniente desativar a criação de alguns recursos com base no espaço de trabalho específico que pretendemos. Para essas ocasiões, podemos usar a variável predefinida terraform.workspace. Esta variável contém o nome da área de trabalho atual e podemos usá-la como qualquer outra em expressões.

# 5. Conclusão
Terraform é uma ferramenta muito poderosa que nos ajuda a adotar a prática de Infraestrutura como Código em nossos projetos. Esse poder, no entanto, vem com seus desafios. Neste artigo, fornecemos uma visão geral rápida dessa ferramenta para que possamos entender melhor seus recursos e seus conceitos básicos.