# Guia Prático: Protegendo seu Servidor FiveM contra SQL Injection

## Introdução
O SQL Injection é uma das vulnerabilidades mais perigosas para servidores que utilizam bancos de dados. No FiveM, falhas de segurança podem permitir que jogadores mal-intencionados manipulem consultas SQL para roubar informações, deletar dados ou comprometer o servidor.

Este guia ensinará como proteger seu servidor FiveM contra SQL Injection de forma eficaz.

---

## 1. O que é SQL Injection?
SQL Injection é uma técnica de ataque onde um invasor insere comandos SQL maliciosos em entradas de usuário para manipular o banco de dados.

**Exemplo de um ataque:**

```lua
local user_id = args[1]
local result = MySQL.query("SELECT * FROM users WHERE id = " .. user_id)
```

Se o jogador digitar `1; DROP TABLE users;`, a tabela `users` será excluída.

---

## 2. Como Proteger-se Contra SQL Injection

### 2.1. Use Queries Preparadas (Prepared Statements)
As consultas preparadas evitam SQL Injection ao separar os dados da lógica SQL.

**Errado:**
```lua
local user_id = args[1]
local result = MySQL.query("SELECT * FROM users WHERE id = " .. user_id)
```

**Correto:**
```lua
local user_id = args[1]
local result = MySQL.query("SELECT * FROM users WHERE id = ?", {user_id})
```
Isso impede que um invasor execute código SQL arbitrário.

### 2.2. Sanitização de Entradas
Antes de processar qualquer entrada do jogador, remova caracteres suspeitos.

```lua
function sanitizeInput(input)
    local sanitized = input:gsub("[\"'%;%-]", "") -- Remove aspas, ponto e vírgula e traço
    return sanitized
end
```
Use essa função antes de passar valores para o banco de dados.

### 2.3. Bloqueio de Padrões Suspeitos
Crie um sistema de detecção de comandos SQL maliciosos.

```lua
function detectSQLInjection(text)
    local patterns = {"UNION", "SELECT", "INSERT", "DELETE", "UPDATE", "DROP", "ALTER", "--", "' OR '1'='1"}
    text = text:upper()
    for _, pattern in pairs(patterns) do
        if text:find(pattern) then
            return true -- SQL Injection detectado
        end
    end
    return false
end
```

Antes de processar entradas do jogador:

```lua
if detectSQLInjection(user_input) then
    DropPlayer(source, "Tentativa de SQL Injection detectada.")
    return
end
```

### 2.4. Logs e Banimento Automático
Registre tentativas de SQL Injection e bana automaticamente os jogadores suspeitos.

```lua
function logSQLInjection(source, input)
    local playerName = GetPlayerName(source)
    local logMessage = ("[SQL INJECTION] Jogador: %s | ID: %d | Entrada suspeita: %s"):format(playerName, source, input)
    PerformHttpRequest("DISCORD_WEBHOOK_URL", function() end, "POST", json.encode({content = logMessage}), { ["Content-Type"] = "application/json" })
    DropPlayer(source, "Você foi banido por tentativa de SQL Injection.")
end

RegisterServerEvent("checkSQLInjection")
AddEventHandler("checkSQLInjection", function(input)
    if detectSQLInjection(input) then
        logSQLInjection(source, input)
    end
end)
```

### 2.5. Restringindo Permissões no Banco de Dados
Crie um usuário específico do MySQL para o FiveM com permissões limitadas.

```sql
CREATE USER 'fivemuser'@'localhost' IDENTIFIED BY 'senha_segura';
GRANT SELECT, INSERT, UPDATE, DELETE ON fivem_database.* TO 'fivemuser'@'localhost';
FLUSH PRIVILEGES;
```
Isso impede que um ataque consiga apagar tabelas do banco de dados.

---

## Conclusão
Implementar proteções contra SQL Injection no seu servidor FiveM é essencial para garantir a segurança dos dados e a estabilidade do servidor.

### **Checklist de Proteção:**
✅ Usar **queries preparadas** (prepared statements)  
✅ **Sanitizar** entradas do jogador  
✅ **Detectar e bloquear** padrões suspeitos  
✅ **Registrar e banir** jogadores que tentam explorar vulnerabilidades  
✅ **Restringir permissões** no banco de dados  

Com essas técnicas, você reduzirá drasticamente o risco de ataques SQL Injection no seu servidor FiveM.

Se precisar de ajuda para implementar isso no seu anti-cheat ou servidor, estou à disposição! 🚀

