# 📚 Documentação de Workflows n8n - Assistentes Financeiros IA

## 🇧🇷 Versão em Português

---

### 📊 Visão Geral dos Workflows

Este conjunto de workflows implementa um sistema completo de assistentes virtuais baseados em IA para consultas financeiras, com foco especial em cotações de moedas estrangeiras.

---

## 1️⃣ Workflow: "Primeiro Fluxo"

### 🎯 Objetivo
Assistente financeiro multilíngue do banco XPTO especializado em cotações de câmbio com integração WhatsApp.

### 🏗️ Arquitetura

```
[Chat Trigger] → [AI Agent] → [Resposta]
                      ↓
                [OpenAI Model]
                      ↓
                [Memory Buffer]
                      ↓
                [Date & Time Tool]
```

### 📦 Componentes

#### 1. **When chat message received** (Chat Trigger)
- **Tipo**: `@n8n/n8n-nodes-langchain.chatTrigger`
- **Configuração**:
  - Modo público habilitado
  - Sessão não suportada
  - Modo de resposta: último nó
- **Webhook ID**: `e4ecc558-d527-4159-b8e9-2ae523fb1785`

#### 2. **AI Agent** (Agente Principal)
- **Tipo**: `@n8n/n8n-nodes-langchain.agent`
- **Prompt do Sistema**:
```
Você é um assistente financeiro do banco XPTO especializado em cotação de moedas estrangeiras.

IDIOMAS: Aceita consultas em qualquer idioma (espanhol, português, inglês, francês, alemão, italiano, chinês, japonês, etc.) e responde automaticamente no mesmo idioma identificado.

FUNÇÃO: Consultar na internet informações atualizadas sobre cotações de hoje para:
- Dólar comercial
- Dólar turismo
- Euro
- Libra esterlina
- Peso argentino
- Peso chileno
- Outras moedas relevantes

FORMATO DE RESPOSTA (quando solicitado cálculo):
| Valor em reais | Moeda consultada | Tipo de cotação | Cotação atual | Valor convertido |
```

#### 3. **OpenAI Chat Model**
- **Modelo**: `gpt-4o-mini`
- **Temperatura**: 0.3 (respostas mais precisas)
- **Ferramentas habilitadas**:
  - Web Search (contexto baixo)
- **Credenciais**: "Aula PMEs"

#### 4. **Simple Memory**
- **Tipo**: Buffer Window Memory
- **Chave de sessão**: `remoteJid` (ID do WhatsApp)
- **Janela de contexto**: 15 mensagens
- **Função**: Mantém histórico da conversa

#### 5. **Date & Time Tool**
- **Função**: Fornece data/hora atual para consultas contextuais
- **Timezone**: Configurável via parâmetros

### 🔌 Integrações

#### WhatsApp via Evolution API
O workflow recebe dados do WhatsApp no seguinte formato:

```json
{
  "chatInput": "mensagem do usuário",
  "sessionId": "{telefone}@s.whatsapp.net",
  "remoteJid": "{telefone}@s.whatsapp.net",
  "pushName": "Nome do Usuário",
  "keyId": "ID_DA_MENSAGEM",
  "fromMe": false,
  "instanceName": "Whatsapp Oficial - MMSoft",
  "serverUrl": "https://evolapi.{domain_name)",
  "apiKey": "CHAVE_API"
}
```

### ⚙️ Configurações do Workflow
- **Status**: Ativo
- **Ordem de execução**: v1
- **Política de chamadas**: workflowsFromSameOwner
- **Timeout**: Ilimitado (-1)
- **MCP**: Desabilitado

### 📝 Casos de Uso
1. Consulta de cotação do dólar hoje
2. Conversão de valores em reais para moeda estrangeira
3. Comparação de cotações comercial vs turismo
4. Histórico de variações cambiais
5. Consultas multilíngues automáticas

---

## 2️⃣ Workflow: "Chatbot"

### 🎯 Objetivo
Nathan - Assistente financeiro especializado focado exclusivamente em cotações do dólar em países da América Latina.

### 🏗️ Arquitetura

```
[Chat Trigger] → [AI Agent] → [Resposta]
                      ↓
                [OpenAI gpt-5-mini]
                      ↓
                [Simple Memory]
                      ↓
                [MCP Client]
```

### 📦 Componentes

#### 1. **When chat message received**
- **Configuração**:
  - Público: Sim
  - Origens permitidas: `*` (todas)
  - Sessão: Não suportada
- **Webhook ID**: `df02626d-b7e1-4238-ad46-dd09842980fd`

#### 2. **AI Agent - Nathan**
- **Personalidade**: Profissional, cordial, especialista em câmbio
- **REGRA FUNDAMENTAL**: Compreende TODOS os idiomas, mas SEMPRE responde em espanhol

**Prompt do Sistema Completo**:
```
Você é Nathan, um assistente financeiro experto e profissional especializado em cotações de moedas.

REGRA FUNDAMENTAL: Compreende mensagens em qualquer idioma (inglês, português, francês, etc.), mas SEMPRE responde em espanhol.

Tarefa principal: Ajudar usuários a consultar a cotação do dólar (USD) em diferentes países da América Latina e do mundo.

FLUXO DE ATENDIMENTO:
1. Saudar cordialmente de forma breve e profissional
2. Perguntar especificamente de qual país precisa a cotação
3. Oferecer opções comuns:
   - Brasil (BRL)
   - Argentina (ARS)
   - México (MXN)
   - Colombia (COP)
   - Chile (CLP)
   - Perú (PEN)
   - Uruguay (UYU)
   - Paraguay (PYG)
   - Outros países sob solicitação

4. Realizar busca na internet para cotação atual
5. Apresentar informação estruturada:
   - Tipo de câmbio atual (compra/venda)
   - Data e hora da cotação
   - Fonte da informação
   - Diferenças mercado oficial vs paralelo/blue (especialmente Argentina)
   - Tendência recente (subiu/baixou)
```

**Exemplo de Interação**:
```
Usuário (qualquer idioma): "What's the dollar price today?"
Nathan: "¡Buen día! Con gusto le ayudo con la cotización del dólar. 
         ¿De qué país necesita conocer la cotización?"

Usuário: "Brasil"
Nathan: "📊 Cotización USD/BRL - Brasil
         📅 Fecha: [atual]
         💵 Dólar Comercial:
         - Compra: R$ X,XX
         - Venta: R$ X,XX
         📈 La cotización ha subido un X% en las últimas 24 horas.
         Fuente: [fonte]"
```

#### 3. **OpenAI Chat Model**
- **Modelo**: `gpt-5-mini` (modelo avançado)
- **Ferramentas integradas**:
  - Web Search (contexto médio)
  - Code Interpreter (habilitado)

#### 4. **Simple Memory**
- **Janela de contexto**: 1 mensagem
- **Função**: Memória mínima para conversas diretas

#### 5. **MCP Client**
- **Endpoint SSE**: `https://webhook.{domain_name)/mcp/mywpBot/sse`
- **Função**: Conexão com servidor MCP para ferramentas externas

### 🌐 Landing Page

A interface web está configurada em: `https://webhook.{domain_name)/webhook/cliente`

**Características**:
- Design responsivo
- Gradiente roxo moderno
- 6 cards de funcionalidades
- Chat integrado n8n
- Mensagens iniciais automáticas em espanhol

### ⚙️ Configurações
- **Status**: Ativo
- **Ordem de execução**: v1
- **Webhook público**: Sim

---

## 3️⃣ Workflow: "HTML"

### 🎯 Objetivo
Landing page profissional para apresentar o assistente virtual com chat integrado.

### 🏗️ Arquitetura

```
[Webhook] → [HTML Generator] → [Respond to Webhook]
```

### 🎨 Design da Landing Page

#### Header
- Logo: 🤖 AsistenteIA
- Fundo: Gradiente roxo (#667eea → #764ba2)

#### Hero Section
```html
Título: "Tu Asistente Virtual Inteligente"
Subtítulo: "Respuestas instantáneas, disponible 24/7, potenciado por IA"
CTA: Botão "Comenzar Conversación"
```

#### Features (6 Cards)

1. **⚡ Respuestas Rápidas**
   - Respostas imediatas em segundos

2. **🌐 Disponible 24/7**
   - Sempre disponível, qualquer dia/hora

3. **🧠 Inteligencia Artificial**
   - IA avançada para resolver necessidades

4. **🔒 Seguro y Privado**
   - Altos padrões de segurança

5. **💬 Conversación Natural**
   - Interação natural como pessoa real

6. **🎯 Soluciones Precisas**
   - Respostas específicas e personalizadas

#### Chat Integration
```javascript
import { createChat } from '@n8n/chat/dist/chat.bundle.es.js';

createChat({
  webhookUrl: 'https://webhook.{domain_name)/webhook/df02626d-b7e1-4238-ad46-dd09842980fd/chat',
  mode: 'window',
  showWelcomeScreen: true,
  initialMessages: [
    '¡Hola! 👋',
    'Mi nombre es Nathan. ¿Cómo puedo ayudarte hoy?'
  ],
  i18n: {
    es: {
      title: 'Inicia una conversación. Estamos aquí para ayudarte 24/7.',
      inputPlaceholder: 'Escribe tu mensaje...',
      getStarted: 'Nueva conversación'
    }
  }
});
```

### 📱 Responsividade
- Mobile-first design
- Grid adaptativo
- Font-size ajustável
- Media queries para tablets e mobile

### 🔗 Endpoints
- **Webhook Path**: `/cliente`
- **Response Mode**: Response Node
- **Headers**: `Content-Type: text/html; charset=utf-8`
- **Origens permitidas**: `*` (CORS aberto)

---

## 4️⃣ Workflow: "MCP Server"

### 🎯 Objetivo
Servidor MCP (Model Context Protocol) que expõe ferramentas do Google Calendar e Supabase para uso por agentes IA.

### 🏗️ Arquitetura

```
[MCP Server Trigger]
         ↓
    [5 Tools AI]
         ├─→ [Google Calendar - Buscar]
         ├─→ [Google Calendar - Criar]
         ├─→ [Google Calendar - Apagar]
         ├─→ [Date & Time]
         └─→ [Supabase]

[Error Trigger] → [Edit Fields] (tratamento de erros)
```

### 📦 Componentes

#### 1. **MCP Server Trigger**
- **Path**: `/mcp/mywpBot`
- **Webhook ID**: `1d98b80e-ea72-429f-813e-b0b34911281b`
- **Função**: Expõe ferramentas via protocolo MCP

#### 2. **Google Calendar - Buscar Horário**
- **Recurso**: Calendar
- **Calendário**: `mmsoft.solucaodigital@gmail.com` (MMSoft Agenda)
- **Parâmetros dinâmicos via `$fromAI`**:
  - `timeMin`: Horário inicial
  - `timeMax`: Horário final
- **Credenciais**: Google Calendar - Alex Marra

#### 3. **Google Calendar - Criar Eventos**
- **Operação**: Create Event
- **Calendário**: MMSoft Agenda
- **Parâmetros AI**:
  - `start`: Data/hora início (formato ISO)
  - `end`: Data/hora fim (formato ISO)
  - `useDefaultReminders`: Boolean
- **Instrução para AI**: "Verifique a data e coloque o nome do evento correto para a agenda"

#### 4. **Google Calendar - Apagar Evento**
- **Operação**: Delete
- **Parâmetro AI**:
  - `eventId`: ID do evento para exclusão

#### 5. **Date & Time Tool**
- **Parâmetros AI**:
  - `includeTime`: Incluir hora atual
  - `outputFieldName`: Nome do campo de saída
  - `timezone`: Fuso horário

#### 6. **Supabase**
- **Operação**: Get All
- **Tabela**: `SMSTable`
- **ReturnAll**: true
- **Retry on Fail**: true
- **Continue on Error**: true
- **Credenciais**: Local Supabase

#### 7. **Error Trigger + Edit Fields**
- Captura erros do workflow
- Processa e formata mensagens de erro
- Pode executar workflow recursivo para tratamento

#### 8. **Execute Workflow**
- Executa o próprio workflow recursivamente
- ID: `rDE1Ij39rtCnxgU3`

### 🔧 Parâmetros `$fromAI`

O sistema `$fromAI` permite que a IA preencha parâmetros dinamicamente:

```javascript
$fromAI('Parameter_Name', 'instruction_hint', 'type')

// Exemplos:
$fromAI('Start_Time', '', 'string')
$fromAI('Use_Default_Reminders', 'Verifique a data...', 'boolean')
$fromAI('Event_ID', '', 'string')
```

### 🌐 Integração MCP Client

Outros workflows conectam-se via **MCP Client**:
```
SSE Endpoint (exemplo): https://webhook.{domain_name)/mcp/mywpBot/sse
```

### ⚙️ Configurações
- **Status**: Ativo
- **Ordem de execução**: v1
- **Template Setup**: Completo

---

## 🔄 Fluxo de Dados Completo

### Cenário: Usuário consulta cotação do dólar

```
1. WhatsApp → Evolution API
   └─> POST para webhook n8n

2. n8n Chat Trigger recebe:
   {
     "chatInput": "Qual a cotação do dólar hoje?",
     "remoteJid": "{telefone}@s.whatsapp.net",
     ...
   }

3. AI Agent (Nathan):
   a) Identifica idioma (português)
   b) Busca memória da sessão
   c) Pergunta qual país (em espanhol)
   
4. Usuário responde: "Brasil"

5. Nathan:
   a) Usa Web Search Tool
   b) Busca cotação USD/BRL atual
   c) Consulta Date & Time para timestamp
   d) Formata resposta em espanhol

6. Resposta enviada:
   "📊 Cotización USD/BRL - Brasil
    📅 Fecha: 21/11/2025 00:53
    💵 Dólar Comercial:
    - Compra: R$ 5,38
    - Venta: R$ 5,38
    💱 Dólar Turismo: R$ 5,53
    📈 Subió un 0,38% en las últimas 24 horas
    Fuente: Banco Central de Brasil"

7. Resposta retorna ao WhatsApp via Evolution API
```

---

## 🚀 Guia de Implantação

### Pré-requisitos

1. **n8n instalado** (self-hosted ou cloud)
2. **Conta OpenAI** com créditos
3. **Evolution API** configurada para WhatsApp
4. **Google Calendar API** habilitada
5. **Supabase** configurado (opcional, para MCP Server)

### Passo a Passo

#### 1. Importar Workflows

```bash
# Via n8n CLI
n8n import:workflow --input=Primeiro_Fluxo.json
n8n import:workflow --input=Chatbot.json
n8n import:workflow --input=HTML.json
n8n import:workflow --input=MCP_Server.json
```

Ou via interface:
1. Settings → Import from File
2. Selecionar cada arquivo JSON
3. Configurar credenciais

#### 2. Configurar Credenciais

**OpenAI**:
```
Nome: Aula PMEs
API Key: sk-...
```

**Google Calendar**:
```
Nome: Google Calendar - Alex Marra
Tipo: OAuth2
Scopes: calendar.readonly, calendar.events
```

**Supabase** (se usar MCP):
```
Nome: Local Supabase
URL: https://seu-projeto.supabase.co
API Key: sua_chave_api
```

#### 3. Configurar Evolution API

No workflow "Primeiro Fluxo", ajustar:
```json
{
  "serverUrl": "https://sua-evolution-api.com.br",
  "apiKey": "SUA_CHAVE_API"
}
```

#### 4. Testar Webhooks

**Chat Trigger**:
```bash
curl -X POST https://seu-n8n.com/webhook/df02626d-b7e1-4238-ad46-dd09842980fd/chat \
  -H "Content-Type: application/json" \
  -d '{"chatInput":"Olá"}'
```

**Landing Page**:
```bash
curl https://seu-n8n.com/webhook/cliente
```

#### 5. Ativar Workflows

Na interface n8n:
1. Abrir cada workflow
2. Toggle "Active" = ON
3. Verificar status de execução

---

## 🔒 Segurança

### Boas Práticas Implementadas

1. **Webhook IDs únicos**: Cada trigger tem ID exclusivo
2. **CORS configurado**: Origens controladas ou `*` para público
3. **Rate limiting**: Considerar implementar no proxy reverso
4. **Credenciais isoladas**: Nunca no código, sempre em vault do n8n
5. **Session IDs**: WhatsApp `remoteJid` como identificador único

### Recomendações Adicionais

```nginx
# Nginx rate limiting
limit_req_zone $binary_remote_addr zone=webhook:10m rate=10r/s;

location /webhook/ {
    limit_req zone=webhook burst=20;
    proxy_pass http://n8n:5678;
}
```

---

## 📊 Monitoramento

### Métricas Importantes

1. **Taxa de sucesso**: Execuções bem-sucedidas / total
2. **Tempo de resposta**: Latência média do AI Agent
3. **Erros**: Logs do Error Trigger no MCP Server
4. **Uso de tokens**: Consumo OpenAI por workflow
5. **Sessões ativas**: Contagem de `remoteJid` únicos

### Logs de Exemplo

```json
{
  "workflow": "Chatbot",
  "execution": "12345",
  "sessionId": "{telefone}@s.whatsapp.net",
  "input": "Qual a cotação do dólar?",
  "language_detected": "pt",
  "response_language": "es",
  "tokens_used": 342,
  "duration_ms": 2450,
  "status": "success"
}
```

---

## 🐛 Troubleshooting

### Problema: AI Agent não responde

**Verificar**:
1. Credenciais OpenAI válidas
2. Quota de tokens não excedida
3. Webhook trigger ativo
4. Logs de execução no n8n

**Solução**:
```bash
# Verificar logs
docker logs n8n-container --tail 100

# Testar OpenAI diretamente
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_KEY" \
  -d '{"model":"gpt-4o-mini","messages":[{"role":"user","content":"test"}]}'
```

### Problema: Memory não funciona

**Causa**: `sessionId` ou `remoteJid` inconsistente

**Solução**:
```javascript
// No AI Agent, verificar expressão:
{{ ($json?.data?.key?.remoteJid == undefined) ? $json.remoteJid : $json.data.key.remoteJid }}

// Deve retornar sempre o mesmo ID por usuário
```

### Problema: MCP Client não conecta

**Verificar**:
1. MCP Server Trigger ativo
2. Endpoint SSE acessível
3. Firewall/proxy não bloqueando SSE

**Teste**:
```bash
curl -N -H "Accept: text/event-stream" \
  https://webhook.{domain_name)/mcp/mywpBot/sse
```

---

## 📈 Otimizações

### Performance

1. **Cache de cotações**: 
   - Implementar Redis para cache de 5 minutos
   - Reduzir chamadas à Web Search

2. **Batch de mensagens**:
   - Agrupar múltiplas perguntas em uma execução
   - Usar Queue Mode no n8n

3. **Modelo adequado**:
   - `gpt-4o-mini` para consultas simples (rápido, barato)
   - `gpt-5-mini` para conversas complexas (mais inteligente)

### Custos

**Estimativa mensal** (1000 conversas/dia):
```
OpenAI (gpt-4o-mini):
- Input: 30k mensagens × 150 tokens × $0.15/1M = $0.67
- Output: 30k mensagens × 500 tokens × $0.60/1M = $9.00
Total: ~$10/mês

Evolution API: $20-50/mês (WhatsApp Business)
n8n Cloud: $0 (self-hosted) ou $20/mês (starter)

TOTAL: $30-80/mês
```

---

## 🎓 Exemplos de Uso Avançado

### 1. Adicionar nova moeda

```javascript
// No prompt do AI Agent, adicionar:
"Suporta também consultas para:
- Iene japonês (JPY)
- Franco suíço (CHF)
- Dólar canadense (CAD)"
```

### 2. Alertas de variação cambial

```javascript
// Novo workflow com Schedule Trigger
Schedule: "0 9,17 * * 1-5" // 9h e 17h, seg-sex

[Schedule] → [Get USD/BRL] → [Compare with Yesterday]
                                      ↓
                            [If > 2% variation]
                                      ↓
                            [Send WhatsApp Alert]
```

### 3. Integrar com CRM

```javascript
// No AI Agent, adicionar tool:
[HTTP Request] → POST https://crm.empresa.com/api/leads
{
  "phone": "{{ $json.remoteJid }}",
  "interest": "cotação cambial",
  "timestamp": "{{ $now }}"
}
```

---

## 🌟 Recursos Futuros

### Roadmap Sugerido

**v1.1** (próximo mês):
- [ ] Gráficos de variação cambial
- [ ] Histórico de cotações (7 dias)
- [ ] Alertas personalizados por usuário

**v1.2** (trimestre):
- [ ] Calculadora de IOF
- [ ] Comparação entre casas de câmbio
- [ ] Previsão de tendência (ML)

**v2.0** (semestre):
- [ ] App mobile nativo
- [ ] Integração Telegram/Facebook
- [ ] Dashboard analytics
- [ ] API pública REST

---

## 📞 Suporte

Para dúvidas ou problemas:

- **Documentação n8n**: https://docs.n8n.io
- **Comunidade**: https://community.n8n.io
- **GitHub Issues**: (adicionar link do seu repo)

---

# 🇪🇸 Versión en Español

---

## 📊 Visión General de los Workflows

Este conjunto de workflows implementa un sistema completo de asistentes virtuales basados en IA para consultas financieras, con enfoque especial en cotizaciones de divisas extranjeras.

---

## 1️⃣ Workflow: "Primeiro Fluxo"

### 🎯 Objetivo
Asistente financiero multilingüe del banco XPTO especializado en cotizaciones de cambio con integración WhatsApp.

### 🏗️ Arquitectura

```
[Chat Trigger] → [AI Agent] → [Respuesta]
                      ↓
                [OpenAI Model]
                      ↓
                [Memory Buffer]
                      ↓
                [Date & Time Tool]
```

### 📦 Componentes

#### 1. **When chat message received** (Disparador de Chat)
- **Tipo**: `@n8n/n8n-nodes-langchain.chatTrigger`
- **Configuración**:
  - Modo público habilitado
  - Sesión no soportada
  - Modo de respuesta: último nodo
- **Webhook ID**: `e4ecc558-d527-4159-b8e9-2ae523fb1785`

#### 2. **AI Agent** (Agente Principal)
- **Tipo**: `@n8n/n8n-nodes-langchain.agent`
- **Prompt del Sistema**:
```
Eres un asistente financiero del banco XPTO especializado en cotización de divisas extranjeras.

IDIOMAS: Aceptas consultas en cualquier idioma (español, portugués, inglés, francés, alemán, italiano, chino, japonés, etc.) y respondes automáticamente en el mismo idioma identificado.

FUNCIÓN: Consultar en internet información actualizada sobre cotizaciones de hoy para:
- Dólar comercial
- Dólar turismo
- Euro
- Libra esterlina
- Peso argentino
- Peso chileno
- Otras monedas relevantes

FORMATO DE RESPUESTA (cuando se solicita cálculo):
| Valor en reales | Moneda consultada | Tipo de cotización | Cotización actual | Valor convertido |
```

#### 3. **OpenAI Chat Model**
- **Modelo**: `gpt-4o-mini`
- **Temperatura**: 0.3 (respuestas más precisas)
- **Herramientas habilitadas**:
  - Web Search (contexto bajo)
- **Credenciales**: "Aula PMEs"

#### 4. **Simple Memory**
- **Tipo**: Buffer Window Memory
- **Clave de sesión**: `remoteJid` (ID de WhatsApp)
- **Ventana de contexto**: 15 mensajes
- **Función**: Mantiene historial de conversación

#### 5. **Date & Time Tool**
- **Función**: Proporciona fecha/hora actual para consultas contextuales
- **Timezone**: Configurable vía parámetros

### 🔌 Integraciones

#### WhatsApp vía Evolution API
El workflow recibe datos de WhatsApp en el siguiente formato:

```json
{
  "chatInput": "mensaje del usuario",
  "sessionId": "{telefone}@s.whatsapp.net",
  "remoteJid": "{telefone}@s.whatsapp.net",
  "pushName": "Nombre del Usuario",
  "keyId": "ID_DEL_MENSAJE",
  "fromMe": false,
  "instanceName": "Whatsapp Oficial - MMSoft",
  "serverUrl": "https://evolapi.{domain_name)",
  "apiKey": "CLAVE_API"
}
```

### ⚙️ Configuraciones del Workflow
- **Estado**: Activo
- **Orden de ejecución**: v1
- **Política de llamadas**: workflowsFromSameOwner
- **Timeout**: Ilimitado (-1)
- **MCP**: Deshabilitado

### 📝 Casos de Uso
1. Consulta de cotización del dólar hoy
2. Conversión de valores en reales a moneda extranjera
3. Comparación de cotizaciones comercial vs turismo
4. Historial de variaciones cambiarias
5. Consultas multilingües automáticas

---

## 2️⃣ Workflow: "Chatbot"

### 🎯 Objetivo
Nathan - Asistente financiero especializado enfocado exclusivamente en cotizaciones del dólar en países de América Latina.

### 🏗️ Arquitectura

```
[Chat Trigger] → [AI Agent] → [Respuesta]
                      ↓
                [OpenAI gpt-5-mini]
                      ↓
                [Simple Memory]
                      ↓
                [MCP Client]
```

### 📦 Componentes

#### 1. **When chat message received**
- **Configuración**:
  - Público: Sí
  - Orígenes permitidos: `*` (todos)
  - Sesión: No soportada
- **Webhook ID**: `df02626d-b7e1-4238-ad46-dd09842980fd`

#### 2. **AI Agent - Nathan**
- **Personalidad**: Profesional, cordial, experto en cambio
- **REGLA FUNDAMENTAL**: Comprende TODOS los idiomas, pero SIEMPRE responde en español

**Prompt del Sistema Completo**:
```
Eres Nathan, un asistente financiero experto y profesional especializado en cotizaciones de divisas.

REGLA FUNDAMENTAL: Comprendes mensajes en cualquier idioma (inglés, portugués, francés, etc.), pero SIEMPRE respondes en español.

Tarea principal: Ayudar a usuarios a consultar la cotización del dólar (USD) en diferentes países de América Latina y del mundo.

FLUJO DE ATENCIÓN:
1. Saludar cordialmente de forma breve y profesional
2. Preguntar específicamente de qué país necesita la cotización
3. Ofrecer opciones comunes:
   - Brasil (BRL)
   - Argentina (ARS)
   - México (MXN)
   - Colombia (COP)
   - Chile (CLP)
   - Perú (PEN)
   - Uruguay (UYU)
   - Paraguay (PYG)
   - Otros países bajo solicitud

4. Realizar búsqueda en internet para cotización actual
5. Presentar información estructurada:
   - Tipo de cambio actual (compra/venta)
   - Fecha y hora de la cotización
   - Fuente de la información
   - Diferencias mercado oficial vs paralelo/blue (especialmente Argentina)
   - Tendencia reciente (subió/bajó)
```

**Ejemplo de Interacción**:
```
Usuario (cualquier idioma): "What's the dollar price today?"
Nathan: "¡Buen día! Con gusto le ayudo con la cotización del dólar. 
         ¿De qué país necesita conocer la cotización?"

Usuario: "Brasil"
Nathan: "📊 Cotización USD/BRL - Brasil
         📅 Fecha: [actual]
         💵 Dólar Comercial:
         - Compra: R$ X,XX
         - Venta: R$ X,XX
         📈 La cotización ha subido un X% en las últimas 24 horas.
         Fuente: [fuente]"
```

#### 3. **OpenAI Chat Model**
- **Modelo**: `gpt-5-mini` (modelo avanzado)
- **Herramientas integradas**:
  - Web Search (contexto medio)
  - Code Interpreter (habilitado)

#### 4. **Simple Memory**
- **Ventana de contexto**: 1 mensaje
- **Función**: Memoria mínima para conversaciones directas

#### 5. **MCP Client**
- **Endpoint SSE**: `https://webhook.{domain_name)/mcp/mywpBot/sse`
- **Función**: Conexión con servidor MCP para herramientas externas

### 🌐 Landing Page

La interfaz web está configurada en (ejemplo): `https://webhook.{domain_name)/webhook/cliente`

**Características**:
- Diseño responsivo
- Gradiente morado moderno
- 6 cards de funcionalidades
- Chat integrado n8n
- Mensajes iniciales automáticos en español

### ⚙️ Configuraciones
- **Estado**: Activo
- **Orden de ejecución**: v1
- **Webhook público**: Sí

---

## 3️⃣ Workflow: "HTML"

### 🎯 Objetivo
Landing page profesional para presentar el asistente virtual con chat integrado.

### 🏗️ Arquitectura

```
[Webhook] → [HTML Generator] → [Respond to Webhook]
```

### 🎨 Diseño de la Landing Page

#### Header
- Logo: 🤖 AsistenteIA
- Fondo: Gradiente morado (#667eea → #764ba2)

#### Hero Section
```html
Título: "Tu Asistente Virtual Inteligente"
Subtítulo: "Respuestas instantáneas, disponible 24/7, potenciado por IA"
CTA: Botón "Comenzar Conversación"
```

#### Features (6 Cards)

1. **⚡ Respuestas Rápidas**
   - Respuestas inmediatas en segundos

2. **🌐 Disponible 24/7**
   - Siempre disponible, cualquier día/hora

3. **🧠 Inteligencia Artificial**
   - IA avanzada para resolver necesidades

4. **🔒 Seguro y Privado**
   - Altos estándares de seguridad

5. **💬 Conversación Natural**
   - Interacción natural como persona real

6. **🎯 Soluciones Precisas**
   - Respuestas específicas y personalizadas

#### Integración del Chat
```javascript
import { createChat } from '@n8n/chat/dist/chat.bundle.es.js';

createChat({
  webhookUrl: 'https://webhook.{domain_name)/webhook/df02626d-b7e1-4238-ad46-dd09842980fd/chat',
  mode: 'window',
  showWelcomeScreen: true,
  initialMessages: [
    '¡Hola! 👋',
    'Mi nombre es Nathan. ¿Cómo puedo ayudarte hoy?'
  ],
  i18n: {
    es: {
      title: 'Inicia una conversación. Estamos aquí para ayudarte 24/7.',
      inputPlaceholder: 'Escribe tu mensaje...',
      getStarted: 'Nueva conversación'
    }
  }
});
```

### 📱 Responsividad
- Diseño mobile-first
- Grid adaptativo
- Font-size ajustable
- Media queries para tablets y móviles

### 🔗 Endpoints
- **Webhook Path**: `/cliente`
- **Response Mode**: Response Node
- **Headers**: `Content-Type: text/html; charset=utf-8`
- **Orígenes permitidos**: `*` (CORS abierto)

---

## 4️⃣ Workflow: "MCP Server"

### 🎯 Objetivo
Servidor MCP (Model Context Protocol) que expone herramientas de Google Calendar y Supabase para uso por agentes IA.

### 🏗️ Arquitectura

```
[MCP Server Trigger]
         ↓
    [5 Tools AI]
         ├─→ [Google Calendar - Buscar]
         ├─→ [Google Calendar - Crear]
         ├─→ [Google Calendar - Borrar]
         ├─→ [Date & Time]
         └─→ [Supabase]

[Error Trigger] → [Edit Fields] (tratamiento de errores)
```

### 📦 Componentes

#### 1. **MCP Server Trigger**
- **Path**: `/mcp/mywpBot`
- **Webhook ID**: `1d98b80e-ea72-429f-813e-b0b34911281b`
- **Función**: Expone herramientas vía protocolo MCP

#### 2. **Google Calendar - Buscar Horario**
- **Recurso**: Calendar
- **Calendario**: `mmsoft.solucaodigital@gmail.com` (MMSoft Agenda)
- **Parámetros dinámicos vía `$fromAI`**:
  - `timeMin`: Horario inicial
  - `timeMax`: Horario final
- **Credenciales**: Google Calendar - Alex Marra

#### 3. **Google Calendar - Crear Eventos**
- **Operación**: Create Event
- **Calendario**: MMSoft Agenda
- **Parámetros AI**:
  - `start`: Fecha/hora inicio (formato ISO)
  - `end`: Fecha/hora fin (formato ISO)
  - `useDefaultReminders`: Boolean
- **Instrucción para AI**: "Verifique a data e coloque o nome do evento correto para a agenda"

#### 4. **Google Calendar - Borrar Evento**
- **Operación**: Delete
- **Parámetro AI**:
  - `eventId`: ID del evento para eliminación

#### 5. **Date & Time Tool**
- **Parámetros AI**:
  - `includeTime`: Incluir hora actual
  - `outputFieldName`: Nombre del campo de salida
  - `timezone`: Zona horaria

#### 6. **Supabase**
- **Operación**: Get All
- **Tabla**: `SMSTable`
- **ReturnAll**: true
- **Retry on Fail**: true
- **Continue on Error**: true
- **Credenciales**: Local Supabase

#### 7. **Error Trigger + Edit Fields**
- Captura errores del workflow
- Procesa y formatea mensajes de error
- Puede ejecutar workflow recursivo para tratamiento

#### 8. **Execute Workflow**
- Ejecuta el propio workflow recursivamente
- ID: `rDE1Ij39rtCnxgU3`

### 🔧 Parámetros `$fromAI`

El sistema `$fromAI` permite que la IA complete parámetros dinámicamente:

```javascript
$fromAI('Parameter_Name', 'instruction_hint', 'type')

// Ejemplos:
$fromAI('Start_Time', '', 'string')
$fromAI('Use_Default_Reminders', 'Verifique la fecha...', 'boolean')
$fromAI('Event_ID', '', 'string')
```

### 🌐 Integración MCP Client

Otros workflows se conectan vía **MCP Client**:
```
SSE Endpoint: https://webhook.{domain_name)/mcp/mywpBot/sse
```

### ⚙️ Configuraciones
- **Estado**: Activo
- **Orden de ejecución**: v1
- **Template Setup**: Completo

---

## 🔄 Flujo de Datos Completo

### Escenario: Usuario consulta cotización del dólar

```
1. WhatsApp → Evolution API
   └─> POST para webhook n8n

2. n8n Chat Trigger recibe:
   {
     "chatInput": "¿Cuál es la cotización del dólar hoy?",
     "remoteJid": "{telefone}@s.whatsapp.net",
     ...
   }

3. AI Agent (Nathan):
   a) Identifica idioma (español)
   b) Busca memoria de la sesión
   c) Pregunta qué país
   
4. Usuario responde: "Argentina"

5. Nathan:
   a) Usa Web Search Tool
   b) Busca cotización USD/ARS actual
   c) Consulta Date & Time para timestamp
   d) Formatea respuesta en español

6. Respuesta enviada:
   "📊 Cotización USD/ARS - Argentina
    📅 Fecha: 21/11/2025 00:53
    💵 Dólar Oficial:
    - Compra: $1.410
    - Venta: $1.430
    💱 Dólar Blue: $1.440
    📈 Subió un 0,35% en las últimas 24 horas
    Fuente: Banco Central de Argentina"

7. Respuesta retorna a WhatsApp vía Evolution API
```

---

## 🚀 Guía de Implementación

### Prerequisitos

1. **n8n instalado** (self-hosted o cloud)
2. **Cuenta OpenAI** con créditos
3. **Evolution API** configurada para WhatsApp
4. **Google Calendar API** habilitada
5. **Supabase** configurado (opcional, para MCP Server)

### Paso a Paso

#### 1. Importar Workflows

```bash
# Vía n8n CLI
n8n import:workflow --input=Primeiro_Fluxo.json
n8n import:workflow --input=Chatbot.json
n8n import:workflow --input=HTML.json
n8n import:workflow --input=MCP_Server.json
```

O vía interfaz:
1. Settings → Import from File
2. Seleccionar cada archivo JSON
3. Configurar credenciales

#### 2. Configurar Credenciales

**OpenAI**:
```
Nombre: Aula PMEs
API Key: sk-...
```

**Google Calendar**:
```
Nombre: Google Calendar - Alex Marra
Tipo: OAuth2
Scopes: calendar.readonly, calendar.events
```

**Supabase** (si usa MCP):
```
Nombre: Local Supabase
URL: https://tu-proyecto.supabase.co
API Key: tu_clave_api
```

#### 3. Configurar Evolution API

En el workflow "Primeiro Fluxo", ajustar:
```json
{
  "serverUrl": "https://tu-evolution-api.com.br",
  "apiKey": "TU_CLAVE_API"
}
```

#### 4. Probar Webhooks

**Chat Trigger**:
```bash
curl -X POST https://tu-n8n.com/webhook/df02626d-b7e1-4238-ad46-dd09842980fd/chat \
  -H "Content-Type: application/json" \
  -d '{"chatInput":"Hola"}'
```

**Landing Page**:
```bash
curl https://tu-n8n.com/webhook/cliente
```

#### 5. Activar Workflows

En la interfaz n8n:
1. Abrir cada workflow
2. Toggle "Active" = ON
3. Verificar estado de ejecución

---

## 🔒 Seguridad

### Buenas Prácticas Implementadas

1. **Webhook IDs únicos**: Cada trigger tiene ID exclusivo
2. **CORS configurado**: Orígenes controlados o `*` para público
3. **Rate limiting**: Considerar implementar en proxy reverso
4. **Credenciales aisladas**: Nunca en el código, siempre en vault de n8n
5. **Session IDs**: WhatsApp `remoteJid` como identificador único

### Recomendaciones Adicionales

```nginx
# Nginx rate limiting
limit_req_zone $binary_remote_addr zone=webhook:10m rate=10r/s;

location /webhook/ {
    limit_req zone=webhook burst=20;
    proxy_pass http://n8n:5678;
}
```

---

## 📊 Monitoreo

### Métricas Importantes

1. **Tasa de éxito**: Ejecuciones exitosas / total
2. **Tiempo de respuesta**: Latencia media del AI Agent
3. **Errores**: Logs del Error Trigger en MCP Server
4. **Uso de tokens**: Consumo OpenAI por workflow
5. **Sesiones activas**: Conteo de `remoteJid` únicos

### Logs de Ejemplo

```json
{
  "workflow": "Chatbot",
  "execution": "12345",
  "sessionId": "{telefone}@s.whatsapp.net",
  "input": "¿Cuál es la cotización del dólar?",
  "language_detected": "es",
  "response_language": "es",
  "tokens_used": 342,
  "duration_ms": 2450,
  "status": "success"
}
```

---

## 🐛 Solución de Problemas

### Problema: AI Agent no responde

**Verificar**:
1. Credenciales OpenAI válidas
2. Cuota de tokens no excedida
3. Webhook trigger activo
4. Logs de ejecución en n8n

**Solución**:
```bash
# Verificar logs
docker logs n8n-container --tail 100

# Probar OpenAI directamente
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_KEY" \
  -d '{"model":"gpt-4o-mini","messages":[{"role":"user","content":"test"}]}'
```

### Problema: Memory no funciona

**Causa**: `sessionId` o `remoteJid` inconsistente

**Solución**:
```javascript
// En AI Agent, verificar expresión:
{{ ($json?.data?.key?.remoteJid == undefined) ? $json.remoteJid : $json.data.key.remoteJid }}

// Debe retornar siempre el mismo ID por usuario
```

### Problema: MCP Client no conecta

**Verificar**:
1. MCP Server Trigger activo
2. Endpoint SSE accesible
3. Firewall/proxy no bloqueando SSE

**Prueba**:
```bash
curl -N -H "Accept: text/event-stream" \
  https://webhook.{domain_name)/mcp/mywpBot/sse
```

---

## 📈 Optimizaciones

### Performance

1. **Cache de cotizaciones**: 
   - Implementar Redis para cache de 5 minutos
   - Reducir llamadas a Web Search

2. **Batch de mensajes**:
   - Agrupar múltiples preguntas en una ejecución
   - Usar Queue Mode en n8n

3. **Modelo adecuado**:
   - `gpt-4o-mini` para consultas simples (rápido, económico)
   - `gpt-5-mini` para conversaciones complejas (más inteligente)

### Costos

**Estimación mensual** (1000 conversaciones/día):
```
OpenAI (gpt-4o-mini):
- Input: 30k mensajes × 150 tokens × $0.15/1M = $0.67
- Output: 30k mensajes × 500 tokens × $0.60/1M = $9.00
Total: ~$10/mes

Evolution API: $0 - Host only
WhatsApp Business:20-50/mes
n8n Cloud: $0 (self-hosted) o $20/mes (starter)

TOTAL: $30-80/mes
```

---

## 🎓 Ejemplos de Uso Avanzado

### 1. Agregar nueva moneda

```javascript
// En el prompt del AI Agent, agregar:
"Soporta también consultas para:
- Yen japonés (JPY)
- Franco suizo (CHF)
- Dólar canadiense (CAD)"
```

### 2. Alertas de variación cambiaria

```javascript
// Nuevo workflow con Schedule Trigger
Schedule: "0 9,17 * * 1-5" // 9h y 17h, lun-vie

[Schedule] → [Get USD/BRL] → [Compare with Yesterday]
                                      ↓
                            [If > 2% variation]
                                      ↓
                            [Send WhatsApp Alert]
```

### 3. Integrar con CRM

```javascript
// En AI Agent, agregar tool:
[HTTP Request] → POST https://crm.empresa.com/api/leads
{
  "phone": "{{ $json.remoteJid }}",
  "interest": "cotización cambial",
  "timestamp": "{{ $now }}"
}
```
## 📞 Soporte

Para dudas o problemas:

- **Documentación n8n**: https://docs.n8n.io
- **Comunidad**: https://community.n8n.io
- **GitHub Issues**: (agregar link de tu repo)
---

## 🎯 Resumen Ejecutivo

### ¿Qué hace este sistema?

Este sistema de workflows n8n implementa asistentes virtuales inteligentes que:

1. **Reciben consultas** vía WhatsApp en cualquier idioma
2. **Identifican el idioma** automáticamente
3. **Responden en español** con información actualizada de cotizaciones
4. **Mantienen contexto** de conversación por usuario
5. **Buscan en internet** datos en tiempo real
6. **Se integran con Google Calendar** para agendamiento
7. **Exponen una landing page** profesional
8. **Escalan vía MCP** para herramientas externas

### ¿Por qué es útil?

- ✅ **Atención 24/7** sin intervención humana
- ✅ **Multilingüe** pero con respuestas consistentes
- ✅ **Datos actualizados** vía web search
- ✅ **Memoria conversacional** por usuario
- ✅ **Extensible** vía MCP protocol
- ✅ **Bajo costo** (~$10/mes en IA)
- ✅ **Fácil implementación** (4 workflows listos)

### ¿Cómo empezar?

1. Importar los 4 JSONs en n8n
2. Configurar credenciales (OpenAI + Google Calendar)
3. Conectar Evolution API
4. Activar workflows
5. ¡Listo para usar!
