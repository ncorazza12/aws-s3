# Pipeline de execução automatizada do Terraform

Este guia descreve, passo a passo, como preparar o repositório no GitHub, configurar o bucket S3 na AWS e ajustar os arquivos do Terraform (`provider.tf` e `vars.tf`) para publicar sua própria página estática.

## Faça fork deste repositório para sua conta do GitHub

1. Crie um fork deste repositório.

## Configure secrets para o seu repositório

1. No seu repositório no GitHub, acesse: `Settings` → `Secrets and variables` → `Actions` → `New repository secret`.
2. Cadastre os secrets abaixo com os valores da sua conta AWS:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - `AWS_SESSION_TOKEN` — necessário se estiver usando credenciais temporárias

> Dica: O parâmetro `AWS_SESSION_TOKEN` é necessário se estiver usando credenciais temporárias.

## Crie o bucket de state do Terraform (backend)

1. No Console da AWS, selecione o serviço S3 e crie o bucket de state.
   - Escolha um nome único global, por exemplo: `aws-s3-tfstate-<seuusuario>`
   - Região: `us-east-1`

> Observação: o bucket de state do backend precisa existir antes do `terraform init`. Em pipelines, garanta que ele já exista (criado manualmente ou por um job controlado).

## Configure seu repositório localmente

1. Clone o repositório localmente.
2. Edite a página:
   - Atualize `app/index.html` com o conteúdo da sua preferência.
   - (Opcional) Atualize `app/error.html` (página de erro).

## Ajustes no Terraform

### provider.tf (backend)

1. Abra o arquivo `terraform/provider.tf` e ajuste o nome do bucket S3 do backend do Terraform (o mesmo criado acima). Exemplo:

```hcl
terraform {

  required_version = "~> 1.12.2"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.8.0"
    }
  }

  backend "s3" {
    bucket       = "aws-s3-tfstate-<seuusuario>"  # SUBSTITUA
    key          = "tfstate"
    region       = "us-east-1"
    use_lockfile = true
  }

}
```

### vars.tf (bucket do site)

1. Abra o arquivo `terraform/vars.tf` e defina um nome único para o bucket do site (será criado pelo Terraform):

```hcl
variable "bucket_name" {
  default = "aws-s3-<seuusuario>"  # SUBSTITUA
}
```

## Faça push das suas alterações para o GitHub

No diretório raiz `aws-s3`, execute os comandos Git para enviar suas alterações ao repositório:

```bash
git add -A
git commit -m "Configuração inicial do Terraform"
git push
```

## Verifique o pipeline no GitHub e valide a aplicação web

1. Acesse a aba `Actions` do seu repositório no GitHub.
2. Verifique se o pipeline foi executado com sucesso.
3. Ao final, o output `aws_s3_bucket_website_endpoint` mostrará a URL do seu site. Algo como:

```
http://<bucket-name>.s3-website-<region>.amazonaws.com
```

4. Acesse essa URL em uma nova aba do navegador e valide se o conteúdo HTML esperado foi carregado.

## (Opcional) Atualizar sua página HTML

- Edite `app/index.html` e/ou `app/error.html`.
- Faça push das alterações.
- Valide o pipeline.
- Verifique as alterações no navegador.

---

Pronto! Com esses passos, você consegue replicar a infraestrutura deste repositório e publicar sua própria página estática no S3.
