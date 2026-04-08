# Como Instalar Java 17 no Windows

## ❌ Problema Identificado

Você tem **Java 8** instalado, mas o projeto precisa de **Java 17**.

```
Versão atual: OpenJDK 1.8.0_482
Versão necessária: Java 17+
```

---

## ✅ Solução: Instalar Java 17

### Opção 1: Via Chocolatey (Mais Rápido)

Se você tem Chocolatey instalado:

```powershell
# Instalar OpenJDK 17
choco install openjdk17

# Ou instalar Oracle JDK 17
choco install oraclejdk17
```

### Opção 2: Download Manual (Recomendado)

#### A. Baixar OpenJDK 17 (Grátis, Open Source)

1. **Acesse**: https://adoptium.net/temurin/releases/
2. **Selecione**:
   - Version: **17 - LTS**
   - Operating System: **Windows**
   - Architecture: **x64**
   - Package Type: **JDK**
   - Format: **.msi** (instalador)
3. **Baixe** o arquivo `.msi`
4. **Execute** o instalador
5. **Marque** a opção "Set JAVA_HOME variable"
6. **Marque** a opção "Add to PATH"
7. **Clique** em "Install"

#### B. Baixar Oracle JDK 17 (Alternativa)

1. **Acesse**: https://www.oracle.com/java/technologies/downloads/#java17
2. **Selecione**: Windows → x64 Installer
3. **Baixe** o arquivo `.exe`
4. **Execute** o instalador
5. Siga as instruções padrão

---

## 🔧 Configurar Variáveis de Ambiente

Após instalar, você precisa configurar as variáveis de ambiente:

### Passo 1: Encontrar o Caminho do Java 17

```powershell
# Procurar instalação do Java
dir "C:\Program Files\Java"
dir "C:\Program Files\Eclipse Adoptium"
dir "C:\Program Files\OpenJDK"
```

O caminho será algo como:
- `C:\Program Files\Eclipse Adoptium\jdk-17.0.10.7-hotspot`
- `C:\Program Files\Java\jdk-17`
- `C:\Program Files\OpenJDK\jdk-17.0.2`

### Passo 2: Configurar JAVA_HOME

**Via PowerShell (Permanente)**:

```powershell
# Substitua pelo caminho correto do seu Java 17
$javaPath = "C:\Program Files\Eclipse Adoptium\jdk-17.0.10.7-hotspot"

# Configurar JAVA_HOME
[Environment]::SetEnvironmentVariable("JAVA_HOME", $javaPath, "Machine")

# Adicionar ao PATH
$currentPath = [Environment]::GetEnvironmentVariable("Path", "Machine")
$newPath = "$javaPath\bin;$currentPath"
[Environment]::SetEnvironmentVariable("Path", $newPath, "Machine")
```

**Via Interface Gráfica**:

1. Pressione `Win + Pause` ou vá em "Sistema" → "Configurações avançadas do sistema"
2. Clique em "Variáveis de Ambiente"
3. Em "Variáveis do sistema", clique em "Novo"
4. Nome: `JAVA_HOME`
5. Valor: `C:\Program Files\Eclipse Adoptium\jdk-17.0.10.7-hotspot` (seu caminho)
6. Clique em "OK"
7. Selecione a variável "Path" e clique em "Editar"
8. Clique em "Novo" e adicione: `%JAVA_HOME%\bin`
9. Mova essa entrada para o topo da lista (importante!)
10. Clique em "OK" em todas as janelas

---

## ✅ Verificar Instalação

**IMPORTANTE**: Feche e abra um **novo terminal PowerShell** (como Administrador se possível).

```powershell
# Verificar versão do Java
java -version

# Deve mostrar algo como:
# openjdk version "17.0.10" 2024-01-16
# OpenJDK Runtime Environment Temurin-17.0.10+7 (build 17.0.10+7)
# OpenJDK 64-Bit Server VM Temurin-17.0.10+7 (build 17.0.10+7, mixed mode, sharing)

# Verificar JAVA_HOME
echo $env:JAVA_HOME

# Deve mostrar:
# C:\Program Files\Eclipse Adoptium\jdk-17.0.10.7-hotspot

# Verificar javac (compilador)
javac -version

# Deve mostrar:
# javac 17.0.10
```

---

## 🚀 Compilar o Projeto Novamente

Após instalar Java 17 e configurar as variáveis:

```powershell
# Voltar ao diretório do projeto
cd C:\Users\ravif\OneDrive\Documentos\github\coletor\controle-arquivo-2

# Limpar build anterior
mvn clean

# Compilar novamente
mvn clean install
```

Agora deve funcionar! 🎉

---

## 🔧 Troubleshooting

### Problema: Ainda mostra Java 8

**Causa**: PATH ainda aponta para Java 8 primeiro

**Solução**:

1. Abra "Variáveis de Ambiente"
2. Edite a variável "Path"
3. **Mova** `%JAVA_HOME%\bin` para o **topo** da lista
4. **Remova** qualquer referência antiga ao Java 8
5. Feche **todos** os terminais
6. Abra um **novo** terminal
7. Teste: `java -version`

### Problema: JAVA_HOME não está definido

```powershell
# Verificar se está definido
echo $env:JAVA_HOME

# Se estiver vazio, configurar manualmente:
$env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-17.0.10.7-hotspot"
$env:Path = "$env:JAVA_HOME\bin;$env:Path"

# Testar
java -version
```

### Problema: Maven ainda usa Java 8

```powershell
# Forçar Maven a usar Java 17
mvn -version

# Se mostrar Java 8, configurar explicitamente:
$env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-17.0.10.7-hotspot"
mvn clean install
```

---

## 📋 Checklist de Validação

- [ ] Java 17 instalado
- [ ] `java -version` mostra versão 17.x.x
- [ ] `javac -version` mostra versão 17.x.x
- [ ] `echo $env:JAVA_HOME` mostra caminho correto
- [ ] `mvn -version` mostra Java version: 17.x.x
- [ ] `mvn clean install` compila sem erros

---

## 🎯 Próximo Passo

Depois que Java 17 estiver instalado e funcionando:

1. Volte ao [GUIA_EXECUCAO_LOCAL.md](GUIA_EXECUCAO_LOCAL.md)
2. Continue do **Passo 4: Compilar o Projeto**

---

**Dica**: Se você trabalha com múltiplos projetos que usam diferentes versões do Java, considere usar um gerenciador de versões como:
- **SDKMAN** (para Windows via WSL)
- **jEnv** (para Windows)
- **Chocolatey** para instalar múltiplas versões lado a lado
