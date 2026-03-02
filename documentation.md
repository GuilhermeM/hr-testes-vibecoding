# Teste de Selecao - Vibecoding

Sistema de avaliacao de candidatos para vaga de vibecoding. Mede habilidades de digitacao e fluencia digital atraves de dois testes sequenciais com resultados salvos no Supabase.

---

## Arquivos

| Arquivo | Descricao |
|---------|-----------|
| `index.html` | Pagina de cadastro do candidato + explicacao das 2 partes |
| `teste-digitacao.html` | Parte 1 - Teste de velocidade e precisao de digitacao |
| `teste-fluencia-digital.html` | Parte 2 - Teste de fluencia digital (6 areas) |

---

## Fluxo do Candidato

```
index.html (cadastro)
    |
    | nome + email via query params
    v
teste-digitacao.html (Parte 1)
    |
    | resultado salvo no Supabase
    | botao "Continuar para Parte 2"
    v
teste-fluencia-digital.html (Parte 2)
    |
    | resultado + respostas detalhadas salvas no Supabase
    v
Tela final: "Teste Concluido! Estamos analisando seus resultados."
```

Os dados do candidato (nome/email) sao passados entre paginas via query params na URL:
`?nome=Fulano&email=fulano@email.com`

---

## Stack Tecnica

- **Frontend**: React 18 (via CDN, com Babel standalone para JSX)
- **Backend/DB**: Supabase (PostgreSQL + API REST)
- **Hospedagem**: Arquivos HTML estaticos (pode ser servido de qualquer lugar)

---

## Supabase

### Configuracao

- **URL**: `https://ttfdmujtekgkaeenvgjw.supabase.co`
- **Projeto**: hr-testes

### Tabelas

#### `resultados_digitacao`

Armazena resultados do teste de digitacao (Parte 1).

| Coluna | Tipo | Descricao |
|--------|------|-----------|
| id | UUID | Chave primaria (auto) |
| nome | TEXT | Nome do candidato |
| email | TEXT | Email do candidato |
| wpm_bruto | INTEGER | Palavras por minuto (bruto) |
| wpm_liquido | INTEGER | Palavras por minuto (descontando erros) |
| cpm | INTEGER | Caracteres por minuto |
| precisao | INTEGER | Percentual de precisao (0-100) |
| erros | INTEGER | Numero de caracteres errados |
| tempo_total | INTEGER | Tempo em segundos |
| vs_media | INTEGER | Comparacao percentual vs media (35 wpm) |
| percentil | INTEGER | Percentil estimado do candidato |
| total_chars | INTEGER | Total de caracteres do texto |
| chars_corretos | INTEGER | Caracteres digitados corretamente |
| texto_usado | TEXT | Texto que foi apresentado |
| texto_digitado | TEXT | Texto que o candidato digitou |
| nivel | TEXT | Classificacao (Iniciante, Basico, Intermediario, Bom, Avancado, Profissional) |
| created_at | TIMESTAMPTZ | Data/hora do registro (auto) |

#### `resultados_fluencia`

Armazena resultados do teste de fluencia digital (Parte 2).

| Coluna | Tipo | Descricao |
|--------|------|-----------|
| id | UUID | Chave primaria (auto) |
| nome | TEXT | Nome do candidato |
| email | TEXT | Email do candidato |
| score_geral | INTEGER | Media geral (0-100) |
| score_digitacao | INTEGER | Score da secao de digitacao |
| score_atalhos | INTEGER | Score da secao de atalhos |
| score_navegador | INTEGER | Score da secao de navegador |
| score_arquivos | INTEGER | Score da secao de arquivos |
| score_logica | INTEGER | Score da secao de logica |
| score_pesquisa | INTEGER | Score da secao de pesquisa |
| nivel | TEXT | Classificacao geral (Excelente, Bom, Regular, Precisa praticar) |
| veredicto | TEXT | Veredicto final (Pronto pra vibecodar, Quase la, Vamos preparar o terreno) |
| respostas_detalhadas | JSONB | JSON com todas as respostas por secao |
| created_at | TIMESTAMPTZ | Data/hora do registro (auto) |

### Formato do campo `respostas_detalhadas`

```json
{
  "shortcuts": [
    {
      "pergunta": "Voce digitou um paragrafo inteiro errado e quer desfazer...",
      "resposta_escolhida": "Ctrl + Z",
      "resposta_correta": "Ctrl + Z",
      "acertou": true
    }
  ],
  "browser": [...],
  "files": [...],
  "logic": [...],
  "search": [...]
}
```

### Politicas de Seguranca (RLS)

Ambas as tabelas tem Row Level Security habilitado com:
- **INSERT** permitido para `anon` (candidatos podem inserir)
- **SELECT** permitido para `anon` (permite leitura publica)

> **Nota**: Para um ambiente de producao, considere restringir o SELECT apenas para usuarios autenticados (admin).

---

## SQL de Setup

Execute no SQL Editor do Supabase:

```sql
-- Tabela: Teste de Digitacao
CREATE TABLE resultados_digitacao (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  nome TEXT NOT NULL,
  email TEXT NOT NULL,
  wpm_bruto INTEGER,
  wpm_liquido INTEGER,
  cpm INTEGER,
  precisao INTEGER,
  erros INTEGER,
  tempo_total INTEGER,
  vs_media INTEGER,
  percentil INTEGER,
  total_chars INTEGER,
  chars_corretos INTEGER,
  texto_usado TEXT,
  texto_digitado TEXT,
  nivel TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE resultados_digitacao ENABLE ROW LEVEL SECURITY;
CREATE POLICY "insert_publico_digitacao" ON resultados_digitacao FOR INSERT TO anon WITH CHECK (true);
CREATE POLICY "select_publico_digitacao" ON resultados_digitacao FOR SELECT TO anon USING (true);

-- Tabela: Teste de Fluencia Digital
CREATE TABLE resultados_fluencia (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  nome TEXT NOT NULL,
  email TEXT NOT NULL,
  score_geral INTEGER,
  score_digitacao INTEGER,
  score_atalhos INTEGER,
  score_navegador INTEGER,
  score_arquivos INTEGER,
  score_logica INTEGER,
  score_pesquisa INTEGER,
  nivel TEXT,
  veredicto TEXT,
  respostas_detalhadas JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE resultados_fluencia ENABLE ROW LEVEL SECURITY;
CREATE POLICY "insert_publico_fluencia" ON resultados_fluencia FOR INSERT TO anon WITH CHECK (true);
CREATE POLICY "select_publico_fluencia" ON resultados_fluencia FOR SELECT TO anon USING (true);
```

---

## Detalhes dos Testes

### Parte 1: Teste de Digitacao

- 5 textos aleatorios (sem acentos) sobre temas de programacao/vibecoding
- Metricas: WPM bruto, WPM liquido, CPM, precisao, erros
- Benchmarks: Iniciante (<20), Basico (20-30), Intermediario (30-45), Bom (45-60), Avancado (60-80), Profissional (80+)
- Media de referencia: 35 WPM
- Anti-cola: paste desabilitado, spellcheck/autocorrect off

### Parte 2: Teste de Fluencia Digital

6 secoes avaliadas:

| Secao | Perguntas | O que avalia |
|-------|-----------|-------------|
| Digitacao | 1 (pratico) | Velocidade e precisao ao digitar |
| Atalhos | 5 | Ctrl+Z, Ctrl+C/V, Alt+Tab, Ctrl+F, Ctrl+S |
| Navegador | 5 | Barra de endereco, DevTools, abas, extensoes |
| Arquivos | 5 | Extensoes, Downloads, organizacao de pastas, .zip |
| Logica | 5 | If/else, sequencia, arrays, erros, variaveis |
| Pesquisa | 3 | Google eficiente, debug, fontes confiaveis |

Veredictos finais:
- **>=80% media**: "Pronto pra vibecodar!"
- **>=55% media**: "Quase la!"
- **<55% media**: "Vamos preparar o terreno primeiro"

---

## Como Usar

### Enviar para candidatos

1. Hospede os 3 arquivos HTML em qualquer servidor (ou abra localmente)
2. Envie o link do `index.html` para o candidato
3. O candidato preenche nome/email e faz os 2 testes automaticamente

### Consultar resultados

1. Acesse o Supabase > Table Editor
2. Veja `resultados_digitacao` para resultados da Parte 1
3. Veja `resultados_fluencia` para resultados da Parte 2
4. Clique em uma linha e expanda `respostas_detalhadas` para ver cada resposta individual

### Consultar via SQL

```sql
-- Ver todos os candidatos e seus scores
SELECT
  rf.nome,
  rf.email,
  rd.wpm_liquido,
  rd.precisao,
  rd.nivel AS nivel_digitacao,
  rf.score_geral,
  rf.nivel AS nivel_fluencia,
  rf.veredicto,
  rf.created_at
FROM resultados_fluencia rf
LEFT JOIN resultados_digitacao rd ON rf.email = rd.email
ORDER BY rf.created_at DESC;

-- Ver respostas detalhadas de um candidato
SELECT nome, email, respostas_detalhadas
FROM resultados_fluencia
WHERE email = 'candidato@email.com';
```
