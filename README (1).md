# 🧩 Como criar Access Keys (API Keys) no MinIO Community Edition (2025+)

A partir das versões mais recentes do **MinIO Community Edition** (ex: `RELEASE.2025-09-07`), o painel web **não permite mais criar chaves de acesso (Access Keys / Service Accounts)**.

Este guia ensina **como gerar credenciais internas (Access Key e Secret Key)** de forma segura usando o **MinIO Client (`mc`)**, mesmo quando o MinIO está rodando em **containers Docker, Portainer ou Swarm**.

---

## ⚙️ Pré-requisitos

- Um servidor com MinIO rodando, por exemplo:

  ```yaml
  image: quay.io/minio/minio:latest
  command: server /data --console-address ":9001"
  ```

- Acesso **shell/terminal** ao container (via Portainer ou `docker exec`)
- Credenciais de administrador (`MINIO_ROOT_USER` e `MINIO_ROOT_PASSWORD`)

---

## 🧭 1. Acessar o container do MinIO

Se estiver usando **Portainer**:

1. Vá em **Containers → minio → Console (ou Exec Shell)**  
2. Selecione **/bin/sh** e clique em **Connect**

Ou, se estiver no terminal do servidor, execute:

```bash
docker exec -it minio sh
```

---

## 🧰 2. Instalar o cliente `mc` dentro do container

O **MinIO Client (`mc`)** é a ferramenta oficial para gerenciar usuários, buckets e credenciais.

Execute dentro do container:

```bash
curl -L https://dl.min.io/client/mc/release/linux-amd64/mc -o /usr/local/bin/mc
chmod +x /usr/local/bin/mc
mc --version
```

Saída esperada:

```
mc version RELEASE.2025-08-13T08-35-41Z
```

---

## 🔗 3. Conectar o `mc` ao servidor MinIO

Crie um alias local para o seu servidor:

```bash
mc alias set local http://localhost:9000 <USUARIO> "<SENHA>"
```

Exemplo:

```bash
mc alias set local http://localhost:9000 minioadmin "minioadmin"
```

Saída esperada:

```
Added `local` successfully.
```

---

## 🔑 4. Criar novas credenciais (Access Key e Secret Key)

### 🔹 Criação automática (geração aleatória)

```bash
mc admin accesskey create local
```

Saída esperada:

```
AccessKey: 1QL4Q4IA64AMP4UM3QWB
SecretKey: A4v6PfQhMHAR50MWIk+PLbJuyh+uynClwOeM+S+X
Expiration: NONE
```

> ⚠️ **Atenção:** copie e guarde as chaves agora.  
> O `SecretKey` **só aparece uma vez** na criação.

---

### 🔹 Criação com nome e senha personalizados

Você pode definir chaves específicas para cada aplicação:

```bash
mc admin accesskey create local --access-key my_app_key --secret-key MySecret123
```

Saída esperada:

```
AccessKey: my_app_key
SecretKey: MySecret123
Expiration: NONE
```

---

## 📋 5. Listar as chaves criadas

Para listar todas as credenciais existentes:

```bash
mc admin accesskey list local
```

Saída esperada:

```
Access Key: my_app_key
Status: enabled
Policy: consoleAdmin
```

---

## 🧪 6. Testar o acesso via protocolo S3

Teste a conexão usando as novas chaves:

```bash
mc alias set s3 https://meu-endpoint-s3.meudominio.com my_app_key "MySecret123"
mc ls s3
```

Se listar os buckets, está tudo funcionando ✅

---

## 🚀 Exemplo de uso em aplicações compatíveis com S3

| Campo | Valor |
|--------|--------|
| **Endpoint** | `https://meu-endpoint-s3.meudominio.com` |
| **Region** | `us-east-1` (ou sua região configurada) |
| **Access Key** | `my_app_key` |
| **Secret Key** | `MySecret123` |
| **Use SSL** | ✅ Sim |

Essas credenciais funcionam com:
- SDKs compatíveis com AWS S3 (Node.js, Python, PHP, etc.)
- APIs de armazenamento (como Evolution API)
- Ferramentas de backup (Rclone, Restic, Cyberduck, etc.)

---

## 🧠 Observações importantes

- A flag `--user` foi **removida nas versões recentes** do `mc`.
- Cada chave criada fica **vinculada ao usuário logado** no `mc alias set`.
- **Community Edition** não mostra mais chaves no painel gráfico (somente via CLI).
- O `SecretKey` **nunca é exibido novamente**, então salve-o ao criar.

---

## 🛠️ Referências oficiais

- Documentação oficial – `mc admin accesskey`: https://min.io/docs/minio/linux/reference/minio-mc-admin/mc-admin-accesskey.html
- Projeto MinIO no GitHub: https://github.com/minio/minio
- Download do MinIO Client (`mc`): https://min.io/download#minio-client

---

## ✨ Autor

Documentação técnica para sysadmins e devops que utilizam **MinIO Community Edition** em ambientes **Docker / Portainer**.

> 💼 Compartilhe e contribua com melhorias no repositório!
