# 🐘🔴 SETUP POSTGRESQL + REDIS

## 📋 ARQUITETURA ATUAL

O sistema utiliza uma arquitetura de três camadas para máxima performance e confiabilidade:

```
PostgreSQL (Database Principal)
    ↓
Redis (Cache de Alta Performance)
    ↓
SQLite (Fallback Local)
```

**Camadas:**
1. **PostgreSQL** - Banco de dados principal em produção
2. **Redis** - Cache em memória para performance 10-100x melhor
3. **SQLite** - Fallback local para desenvolvimento/testes

---

## ✅ SOLUÇÃO 1: Rodar com Docker (RECOMENDADO)

Se você está usando containers Docker, rode o app dentro da mesma rede Docker:

### 1. Usar docker-compose.yml:

O arquivo já existe no projeto com a configuração completa:

```bash
docker-compose up -d
```

### 2. Verificar serviços rodando:

```bash
docker-compose ps
```

Deve mostrar:
- `agenteauzap` (app)
- `pange_pangeia_post` (PostgreSQL)
- `pange_pangeia_redis` (Redis)

### 3. Ver logs:

```bash
docker-compose logs -f agenteauzap
```

---

## ✅ SOLUÇÃO 2: Desenvolvimento Local

Se o PostgreSQL e Redis estão rodando **localmente** (não no Docker), altere o .env:

### Opção A: PostgreSQL e Redis locais

```bash
# .env
DATABASE_URL=postgres://postgres:sua_senha@localhost:5432/seu_database
REDIS_URL=redis://default:sua_senha@localhost:6379
```

### Opção B: PostgreSQL no servidor remoto

```bash
# .env
DATABASE_URL=postgres://postgres:senha@IP_DO_SERVIDOR:5432/database
REDIS_URL=redis://default:senha@IP_DO_SERVIDOR:6379
```

### Opção C: PostgreSQL gerenciado (Railway, Render, etc.)

```bash
# .env
DATABASE_URL=postgres://user:pass@host.railway.app:5432/railway
REDIS_URL=redis://default:pass@redis.railway.app:6379
```

---

## ✅ SOLUÇÃO 3: Apenas SQLite (Desenvolvimento Local)

Para desenvolvimento rápido sem configurar PostgreSQL/Redis:

```bash
# .env
# Comentar ou remover DATABASE_URL e REDIS_URL
# DATABASE_URL=...
# REDIS_URL=...
```

O sistema usará SQLite automaticamente como fallback.

---

## 🧪 COMO TESTAR CONEXÕES

### Test PostgreSQL:

```bash
# Dentro do Docker:
docker exec -it pange_pangeia_post psql -U postgres -d pange

# Local:
psql "postgres://postgres:senha@localhost:5432/database"

# Verificar tabelas:
\dt
```

### Test Redis:

```bash
# Dentro do Docker:
docker exec -it pange_pangeia_redis redis-cli -a sua_senha

# Local:
redis-cli -h localhost -p 6379 -a sua_senha

# Testar:
PING
# Deve retornar: PONG
```

### Test SQLite:

```bash
# Ver arquivo do banco:
ls -lh data/customers.db

# Abrir no SQLite:
sqlite3 data/customers.db
.tables
.schema user_profiles
```

---

## 📊 STATUS ESPERADO NO STARTUP

### ✅ PostgreSQL + Redis (Performance Máxima):

```
🚀 Iniciando Agente Pet Shop WhatsApp...

✅ PostgreSQL conectado com sucesso (DATABASE_URL)
   Host: pange_pangeia_post (ou localhost)
🐘 Testando conexão PostgreSQL...
✅ PostgreSQL: Conexão verificada e funcionando!
   Server time: 2025-01-18 18:33:45.123

✅ Redis conectado com sucesso
✅ Redis pronto para uso
🔴 Testando conexão Redis...
✅ Redis: Conexão testada com sucesso

📊 CustomerMemoryDB: POSTGRESQL + REDIS CACHE
   ✅ Performance máxima com cache
   ✅ Queries 10-100x mais rápidas
```

### ⚠️ PostgreSQL sem Redis:

```
✅ PostgreSQL conectado com sucesso
   Host: localhost

ℹ️  REDIS_URL não configurado - cache desabilitado

📊 CustomerMemoryDB: POSTGRESQL (sem cache)
   💡 Configure REDIS_URL para melhor performance
```

### ℹ️ Fallback SQLite (Desenvolvimento):

```
ℹ️  DATABASE_URL não configurado - usando SQLite local
ℹ️  REDIS_URL não configurado - cache desabilitado

📊 CustomerMemoryDB: SQLITE (fallback local)
   💡 Configure DATABASE_URL e REDIS_URL para produção
   📁 Dados salvos em: /app/data/customers.db
```

---

## 🚀 RECOMENDAÇÃO POR AMBIENTE

### 🏭 PRODUÇÃO (Render, Railway, VPS)

**Use PostgreSQL + Redis:**
```bash
DATABASE_URL=postgres://user:pass@host:5432/database
REDIS_URL=redis://default:pass@host:6379
```

**Serviços gerenciados recomendados:**
- PostgreSQL: Railway, Render, Neon, DigitalOcean
- Redis: Upstash, Redis Cloud, Railway

### 🐳 DOCKER LOCAL

**Use docker-compose.yml:**
```bash
docker-compose up -d
```

Todos os serviços na mesma rede, hostnames funcionam automaticamente.

### 💻 DESENVOLVIMENTO LOCAL

**Opção 1 - SQLite (mais simples):**
```bash
# Sem configurar nada, usa fallback local
npm start
```

**Opção 2 - PostgreSQL local:**
```bash
# Instalar PostgreSQL e Redis localmente
brew install postgresql redis  # macOS
sudo apt install postgresql redis  # Linux

# Configurar .env
DATABASE_URL=postgres://postgres:senha@localhost:5432/auzap
REDIS_URL=redis://localhost:6379
```

---

## 🔧 CONFIGURAÇÃO DE PRODUÇÃO

### 1. PostgreSQL

**Criar database:**
```sql
CREATE DATABASE auzap;
```

**Executar migrations:**
```sql
-- O sistema cria tabelas automaticamente no primeiro boot
-- Mas você pode executar manualmente se preferir:

CREATE TABLE IF NOT EXISTS user_profiles (
    phone TEXT PRIMARY KEY,
    name TEXT,
    email TEXT,
    pet_name TEXT,
    pet_type TEXT,
    pet_breed TEXT,
    interests TEXT,
    last_purchase TEXT,
    interaction_count INTEGER DEFAULT 0,
    total_revenue REAL DEFAULT 0,
    created_at TEXT DEFAULT CURRENT_TIMESTAMP,
    updated_at TEXT DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_profiles_email ON user_profiles(email);
CREATE INDEX idx_user_profiles_pet_type ON user_profiles(pet_type);
CREATE INDEX idx_user_profiles_updated_at ON user_profiles(updated_at);
```

### 2. Redis

**Configurar senha (recomendado):**
```bash
# redis.conf
requirepass sua_senha_forte
```

**Testar:**
```bash
redis-cli -a sua_senha_forte
PING
```

### 3. Variáveis de Ambiente

**Mínimo para produção:**
```bash
DATABASE_URL=postgres://user:pass@host:5432/auzap
REDIS_URL=redis://default:pass@host:6379
OPENAI_API_KEY=sk-proj-...
PORT=3000
```

---

## 📊 MONITORAMENTO

### PostgreSQL Stats

```sql
-- Ver tamanho do banco
SELECT pg_size_pretty(pg_database_size('auzap'));

-- Ver número de registros
SELECT COUNT(*) FROM user_profiles;

-- Ver últimas atualizações
SELECT phone, name, updated_at
FROM user_profiles
ORDER BY updated_at DESC
LIMIT 10;
```

### Redis Stats

```bash
redis-cli -a senha INFO stats
redis-cli -a senha INFO memory
redis-cli -a senha KEYS "customer:*"
```

### SQLite Stats

```bash
# Tamanho do arquivo
ls -lh data/customers.db

# Número de registros
sqlite3 data/customers.db "SELECT COUNT(*) FROM user_profiles;"
```

---

## 🆘 TROUBLESHOOTING

### "Cannot find name pange_pangeia_post"
→ Hostname não existe. Use `localhost` ou IP do servidor, ou rode via Docker.

### "Connection refused"
→ PostgreSQL/Redis não está rodando. Inicie os serviços.

### "Password authentication failed"
→ Senha incorreta no DATABASE_URL.

### "CustomerMemoryDB: SQLITE (fallback)"
→ DATABASE_URL não configurado ou conexão falhou.
→ Sistema usa SQLite como fallback (funciona, mas só local).

### "REDIS_URL não configurado"
→ Sistema funciona sem Redis, mas performance será menor.
→ Configure REDIS_URL para cache e performance 10-100x melhor.

### Performance lenta
→ Verifique se Redis está configurado
→ Verifique se PostgreSQL tem índices
→ Use EXPLAIN ANALYZE nas queries lentas

---

## 💡 PRÓXIMOS PASSOS

1. **Escolha seu ambiente:**
   - Produção: PostgreSQL + Redis gerenciados
   - Docker: docker-compose.yml
   - Desenvolvimento: SQLite (fallback)

2. **Configure as variáveis:**
   - DATABASE_URL (PostgreSQL)
   - REDIS_URL (Redis)

3. **Teste as conexões:**
   - Use os comandos acima para verificar

4. **Inicie o sistema:**
   ```bash
   npm start
   # ou
   docker-compose up -d
   ```

5. **Verifique os logs:**
   - Deve mostrar PostgreSQL + Redis conectados
   - Performance máxima com cache

---

## 📚 ARQUIVOS RELACIONADOS

- **docker-compose.yml** - Configuração Docker completa
- **src/services/customerMemoryDB.ts** - Implementação do sistema
- **CONFIGURAR_RENDER_AGORA.md** - Deploy em produção
- **.env.example** - Template de variáveis

---

**Criado**: Janeiro 2025
**Arquitetura**: PostgreSQL (database) + Redis (cache) + SQLite (fallback)
**Performance**: 10-100x melhor com cache Redis
