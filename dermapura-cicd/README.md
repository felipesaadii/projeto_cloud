# DermaPura - Docker + GitHub Actions + Docker Hub + AWS EC2

Projeto para o LAB11. O workflow faz build da imagem Docker, publica no Docker Hub
e conecta por SSH à instância EC2 para atualizar o container que serve a One-Page.

## Estrutura
- `index.html`: One-Page do Case 3 (DermaPura).
- `Dockerfile`: imagem Nginx que publica o HTML na porta 80.
- `.github/workflows/deploy.yml`: pipeline CI/CD.

## 1. Preparar a EC2 (Amazon Linux 2023)

Crie/tenha uma instância EC2 com acesso HTTP (TCP 80) e SSH (TCP 22).
No Security Group, para SSH prefira "My IP"; para HTTP use `0.0.0.0/0`.

Conecte-se por SSH e execute:

```bash
sudo dnf update -y
sudo dnf install docker -y
sudo systemctl enable --now docker
sudo usermod -aG docker ec2-user
```

Depois, encerre a conexão SSH e conecte novamente para atualizar os grupos.
Confirme que o Docker está instalado:

```bash
docker --version
```

O workflow usa `sudo docker`, então a associação ao grupo é opcional para o deploy,
mas é útil para executar Docker manualmente sem sudo.

## 2. Criar imagem e repositório no Docker Hub

1. Crie/entre em https://hub.docker.com/.
2. Crie um repositório chamado `dermapura`.
3. Deixe-o **Public** para que a EC2 possa baixar a imagem sem credenciais.
4. Gere um Access Token no Docker Hub (Account Settings > Personal access tokens).

## 3. Colocar os arquivos no GitHub

Envie para a raiz de um repositório GitHub:
- `index.html`
- `Dockerfile`
- `.dockerignore`
- pasta `.github/workflows/deploy.yml`

O branch deve se chamar `main`.

## 4. Configurar Secrets no GitHub

No repositório: Settings > Secrets and variables > Actions > New repository secret.

Cadastre estes 4 secrets:

| Nome | Valor |
|---|---|
| `DOCKERHUB_USERNAME` | Seu usuário do Docker Hub |
| `DOCKERHUB_TOKEN` | Access Token do Docker Hub (não a senha da conta) |
| `EC2_HOST` | IPv4 público ou DNS público da instância EC2 |
| `EC2_SSH_KEY` | Conteúdo completo do arquivo `.pem` da chave da EC2 |

O usuário SSH foi definido como `ec2-user` no workflow, adequado ao Amazon Linux.
Se sua instância usar outra distribuição, altere `username` no workflow.

**Nunca** coloque token, senha ou arquivo `.pem` no código, no HTML ou no repositório.

## 5. Executar o CI/CD

Faça um commit e push para `main`. Em GitHub > Actions, abra
"CI-CD - DermaPura na AWS EC2". O pipeline deve:
1. baixar o repositório;
2. construir a imagem Docker;
3. publicar `usuario/dermapura:latest` e uma tag com o SHA do commit no Docker Hub;
4. conectar à EC2 por SSH, baixar a imagem e recriar o container `dermapura`.

Acesse no navegador:

```text
http://IP_PUBLICO_DA_EC2
```

## 6. Teste local opcional

Na pasta do projeto:

```bash
docker build -t dermapura:teste .
docker run --rm -d --name dermapura-teste -p 8080:80 dermapura:teste
```

Abra `http://localhost:8080`. Para parar:

```bash
docker stop dermapura-teste
```

## Observações importantes

- O workflow substitui o container anterior a cada publicação; a página fica indisponível por alguns segundos durante a troca.
- O endereço IPv4 público da EC2 pode mudar se a instância for parada e iniciada novamente, a menos que você configure um Elastic IP (que pode ter custo).
- A imagem Docker Hub foi configurada como pública para simplificar o `docker pull` na EC2.
- Este exemplo publica uma página estática. Não contém backend ou banco de dados.
- EC2, armazenamento, tráfego e IPv4 público podem gerar custos. Verifique a estimativa e os limites da sua conta AWS.
