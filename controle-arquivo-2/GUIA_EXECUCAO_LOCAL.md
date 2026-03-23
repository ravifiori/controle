# Guia de Execução Local - Controle de Arquivos

Este guia mostra como executar o sistema completo localmente no seu ambiente Windows.

## Pré-requisitos

Antes de começar, você precisa ter instalado:

1. ✅ **Java 17+** (você já tem)
2. ✅ **Docker Desktop** (você já tem)
3. ❌ **Maven 3.8+** (precisa instalar)

---

## Passo 1: Instalar Maven

### Opção A: Via Chocolatey (Recomendado)

Se você tem Chocolatey instalado:

```powershell
choco install maven
```

### Opção B: Instalação Manual

1. **Baixar Maven**:
   - Acesse: https://maven.apache.org/download.cgi
   - Baixe o arquivo `apache-maven-3.9.6-bin.zip`

2. **Extrair**:
   - Extraia para: `C:\Program Files\Apache\maven`

3. **Configurar PATH**:
   ```powershell
   # Adicionar ao PATH do sistema
   [Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Apache\maven\bin", "Machine")
   ```

4. **Verificar instalação** (abra um novo terminal):
   ```powershell
   mvn --version
   ```

---

## Passo 2: Iniciar Infraestrutura Local

### 2.1 Iniciar Docker Compose

```powershell
# No diretório raiz do projeto
docker-compose up -d
```

Isso iniciará:
- **Oracle XE** (porta 1521)
- **RabbitMQ** (porta 5672, management 15672)
- **LocalStack S3** (porta 4566)
- **SFTP Server** (porta 2222)

### 2.2 Verificar Status dos Serviços

```powershell
docker-compose ps
```

Aguarde até que todos os serviços estejam com status `healthy` (pode levar 2-3 minutos).

### 2.3 Verificar Logs (Opcional)

```powershell
# Ver logs de todos os serviços
docker-compose logs -f

# Ver logs de um serviço específico
docker-compose logs -f oracle
docker-compose logs -f rabbitmq
```

---

## Passo 3: Aplicar Scripts DDL no Oracle

### 3.1 Conectar ao Oracle

Você pode usar qualquer cliente SQL. Exemplos:

**Via SQL*Plus (se instalado)**:
```powershell
sqlplus system/Oracle123@localhost:1521/XE
```

**Via DBeaver / SQL Developer**:
- Host: `localhost`
- Port: `1521`
- SID: `XE`
- Username: `system`
- Password: `Oracle123`

### 3.2 Executar Scripts DDL

Execute os scripts na seguinte ordem:

```sql
-- 1. Criar sequences
@scripts/ddl/01-create-sequences.sql

-- 2. Criar tabelas
@scripts/ddl/02-create-tables.sql

-- 3. Criar índices
@scripts/ddl/03-create-indexes.sql

-- 4. Criar constraints
@scripts/ddl/04-create-constraints.sql

-- 5. Inserir dados de teste (opcional)
@scripts/ddl/05-insert-test-data.sql
```

---

## Passo 4: Compilar o Projeto

```powershell
# No diretório raiz do projeto
mvn clean install
```

Isso irá:
- Compilar todos os módulos (common, orchestrator, processor)
- Executar testes unitários
- Gerar os JARs

**Tempo estimado**: 5-10 minutos (primeira vez)

---

## Passo 5: Executar o Orquestrador

Abra um terminal e execute:

```powershell
cd orchestrator
mvn spring-boot:run -Dspring-boot.run.profiles=local
```

Aguarde até ver a mensagem:
```
Started OrquestradorApplication in X.XXX seconds
```

### Verificar Health do Orquestrador

Em outro terminal:
```powershell
curl http://localhost:8080/actuator/health
```

Resposta esperada:
```json
{
  "status": "UP"
}
```

---

## Passo 6: Executar o Processador

Abra **outro terminal** e execute:

```powershell
cd processor
mvn spring-boot:run -Dspring-boot.run.profiles=local
```

Aguarde até ver a mensagem:
```
Started ProcessadorApplication in X.XXX seconds
```

### Verificar Health do Processador

Em outro terminal:
```powershell
curl http://localhost:8081/actuator/health
```

Resposta esperada:
```json
{
  "status": "UP"
}
```

---

## Passo 7: Testar o Fluxo End-to-End

### 7.1 Colocar Arquivo no SFTP

```powershell
# Conectar ao SFTP
sftp -P 2222 sftpuser@localhost
# Senha: sftppass

# Fazer upload de um arquivo de teste
put test-data/files/cielo/CIELO_20240115.txt upload/

# Sair
exit
```

### 7.2 Acompanhar Processamento

**Logs do Orquestrador**:
- Verá mensagens de coleta do arquivo
- Registro no banco de dados
- Publicação na fila RabbitMQ

**Logs do Processador**:
- Consumo da mensagem
- Download do arquivo
- Identificação de cliente e layout
- Upload para destino (S3 ou SFTP)
- Rastreabilidade completa

### 7.3 Verificar Rastreabilidade no Banco

```sql
-- Ver arquivo coletado
SELECT * FROM file_origin 
WHERE des_file_name = 'CIELO_20240115.txt';

-- Ver processamento
SELECT * FROM file_origin_client_processing 
WHERE idt_file_origin_client = (
  SELECT idt_file_origin_client 
  FROM file_origin_client 
  WHERE idt_file_origin = <id_do_arquivo>
);
```

### 7.4 Verificar Destino

**Se destino for S3 (LocalStack)**:
```powershell
aws --endpoint-url=http://localhost:4566 s3 ls s3://controle-arquivos/
```

**Se destino for SFTP**:
- Conectar ao SFTP e verificar diretório de destino

---

## Passo 8: Acessar Interfaces Web

### RabbitMQ Management

- URL: http://localhost:15672
- Username: `admin`
- Password: `admin123`

Aqui você pode:
- Ver filas e mensagens
- Monitorar throughput
- Ver conexões ativas

### Oracle Enterprise Manager (Opcional)

- URL: http://localhost:5500/em
- Username: `system`
- Password: `Oracle123`

---

## Comandos Úteis

### Parar Tudo

```powershell
# Parar aplicações Spring Boot
# Pressione Ctrl+C nos terminais do Orquestrador e Processador

# Parar infraestrutura Docker
docker-compose down
```

### Reiniciar Infraestrutura

```powershell
docker-compose restart
```

### Ver Logs em Tempo Real

```powershell
# Orquestrador
cd orchestrator
mvn spring-boot:run -Dspring-boot.run.profiles=local

# Processador (em outro terminal)
cd processor
mvn spring-boot:run -Dspring-boot.run.profiles=local
```

### Limpar Dados

```powershell
# Parar tudo
docker-compose down

# Remover volumes (apaga dados do banco)
docker-compose down -v

# Iniciar novamente
docker-compose up -d
```

---

## Troubleshooting

### Problema: Porta já em uso

**Erro**: `Bind for 0.0.0.0:1521 failed: port is already allocated`

**Solução**:
```powershell
# Ver o que está usando a porta
netstat -ano | findstr :1521

# Parar o processo ou mudar a porta no docker-compose.yml
```

### Problema: Oracle não inicia

**Solução**:
```powershell
# Ver logs do Oracle
docker-compose logs oracle

# Reiniciar apenas o Oracle
docker-compose restart oracle
```

### Problema: Orquestrador não coleta arquivos

**Verificações**:
1. Scheduler está habilitado? (Verifique `application-local.yml`)
2. Arquivos estão no diretório correto do SFTP?
3. Configurações estão cadastradas no banco?

### Problema: Processador não consome mensagens

**Verificações**:
1. RabbitMQ está rodando? `docker-compose ps rabbitmq`
2. Fila existe? Acesse http://localhost:15672
3. Credenciais estão corretas?

---

## Próximos Passos

Após rodar localmente com sucesso:

1. **Executar Testes**:
   ```powershell
   mvn clean test
   ```

2. **Verificar Cobertura**:
   ```powershell
   mvn clean test jacoco:report
   # Relatório em: target/site/jacoco/index.html
   ```

3. **Executar Testes de Integração**:
   ```powershell
   mvn verify
   ```

4. **Preparar para Deploy**:
   - Revisar configurações de produção
   - Configurar Secrets no Kubernetes
   - Aplicar manifests K8s

---

## Referências

- [README.md](README.md) - Visão geral do sistema
- [docs/CONFIGURATION.md](docs/CONFIGURATION.md) - Configurações detalhadas
- [docs/OPERATIONS.md](docs/OPERATIONS.md) - Guia de operações
- [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) - Guia de desenvolvimento
- [k8s/README.md](k8s/README.md) - Deploy Kubernetes

---

## Suporte

Se encontrar problemas:
1. Verifique os logs dos serviços
2. Consulte a seção de Troubleshooting
3. Revise a documentação em `docs/`
4. Crie uma issue no repositório
