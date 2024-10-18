## Desafio Terraform: Provisionamento de Infraestrutura AWS

### 1. Descrição Técnica

#### 1.1 Análise Técnica do Código Original
O código Terraform original provê uma infraestrutura básica na AWS com os seguintes recursos:
- **VPC (AWS VPC):** Criação de uma VPC personalizada com o bloco de CIDR `10.0.0.0/16`, habilitando DNS.
- **Subnet (AWS Subnet):** Uma subnet é criada dentro da VPC com o bloco de CIDR `10.0.1.0/24`.
- **Internet Gateway (AWS Internet Gateway):** Um gateway de internet é criado para permitir o tráfego de saída da VPC.
- **Tabela de Rotas (AWS Route Table):** Define uma rota para o tráfego externo (`0.0.0.0/0`) através do Internet Gateway.
- **Associação de Tabela de Rotas (AWS Route Table Association):** A tabela de rotas é associada à subnet criada.
- **Security Group (AWS Security Group):** Configura permissões de entrada para permitir o tráfego SSH (porta 22) de qualquer endereço IP (`0.0.0.0/0`), e todo o tráfego de saída é permitido.
- **Par de Chaves SSH (AWS Key Pair):** Gera uma chave pública RSA para ser usada com a instância EC2.
- **Instância EC2 (AWS EC2):** Provisiona uma instância EC2 do tipo `t2.micro` executando Debian 12, com 20 GB de armazenamento, e associada ao security group criado.

#### 1.2 Melhorias Aplicadas
- **Segurança:**
  - Modifiquei o Security Group para limitar o acesso SSH a uma lista de IPs confiáveis, usando a variável `allowed_ssh_ips`.
  - Adicionei a variável `ssh_key_name` para permitir o uso de um par de chaves SSH já existente ou personalizado.

- **Automação do Nginx:**
  - Adicionei a automação da instalação do servidor Nginx na instância EC2 através de um script `user_data`. O Nginx é instalado e configurado para iniciar automaticamente após a inicialização da instância.

- **Variáveis Configuráveis:**
  - Adicionei a variável `instance_type` para permitir que o tipo de instância EC2 seja configurado de acordo com a necessidade.

---

### 2. Instruções de Uso

#### 2.1 Pré-requisitos
Antes de aplicar a configuração, é necessário:
1. **AWS CLI Configurada:** O Terraform utilizará as credenciais e a configuração da AWS CLI. Certifique-se de que as credenciais estão configuradas corretamente.
2. **Chave SSH:** Uma chave SSH deve ser configurada previamente no seu perfil AWS ou passada como argumento através da variável `ssh_key_name`.
3. **Terraform Instalado:** Certifique-se de que o Terraform está instalado e funcionando na sua máquina.

#### 2.2 Aplicação do Código
1. Clone o repositório ou salve o código em um arquivo chamado `main.tf`.
2. Execute os comandos a seguir para inicializar e aplicar o Terraform:

```bash
# Inicializar o Terraform (instalar plugins necessários)
terraform init

# Planejar a infraestrutura a ser criada
terraform plan

# Aplicar a infraestrutura na AWS
terraform apply
```

3. Durante a execução do comando `apply`, será solicitado que você confirme a aplicação das mudanças. Digite `yes` para confirmar.

#### 2.3 Variáveis Configuráveis
Você pode modificar as variáveis no código ou passar valores através de um arquivo `terraform.tfvars` ou diretamente via linha de comando:
- **projeto:** Nome do projeto.
- **candidato:** Nome do candidato.
- **allowed_ssh_ips:** Lista de IPs permitidos para acessar via SSH.
- **instance_type:** Tipo da instância EC2 (padrão: `t2.micro`).
- **ssh_key_name:** Nome do par de chaves SSH existente.

Exemplo de arquivo `terraform.tfvars`:

```hcl
projeto = "MeuProjeto"
candidato = "Elison"
allowed_ssh_ips = ["203.0.113.0/24"]
ssh_key_name = "minha-chave-ssh"
instance_type = "t3.micro"
```

#### 2.4 Outputs
Ao final da execução, o Terraform exibirá os seguintes outputs:
- **Chave Privada:** A chave privada gerada pelo Terraform, necessária para acessar a instância EC2 via SSH.
- **IP Público da Instância EC2:** O endereço IP público da instância EC2 provisionada.

Você pode acessar a instância EC2 usando o comando SSH a seguir (substitua `<ec2_public_ip>` pelo IP público da instância):

```bash
ssh -i caminho/para/minha-chave.pem ec2-user@<ec2_public_ip>
```

---

### 3. Justificativas das Melhorias
- **Segurança:** A abertura de SSH para qualquer IP é um risco de segurança. Ao limitar o acesso a IPs confiáveis, aumentamos a segurança do ambiente.
- **Automação com Nginx:** A automação da instalação do Nginx simplifica o processo e evita a necessidade de configuração manual após a criação da instância.
- **Flexibilidade:** A adição de variáveis configuráveis permite que o código seja facilmente adaptado para diferentes cenários, sem a necessidade de alterações manuais no código fonte.