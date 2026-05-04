# Run50+ — Sistema Avançado de Treinamento

> Single-file HTML · localStorage · API Anthropic opcional · Modo offline

---

## Visão Geral

**Run50+** é um sistema de gestão de treinos pessoais construído como arquivo HTML único, sem dependências de servidor. Combina planejamento de corrida progressiva com academia e Tabata, avaliação automática de desempenho por score semanal e geração de treinos personalizados por IA.

**Objetivos embutidos no sistema:**
- 🏃 **21 km** em 27/12/2026
- 🏆 **42 km** em 27/12/2027

---

## Como Usar

### Instalação

Nenhuma. Abra `run50plus.html` diretamente no navegador.

Na primeira abertura, o sistema solicita a chave da API Anthropic (opcional). Para pular e usar no modo offline, clique em **"Usar sem IA"**.

### Fluxo diário

```
1. Abrir o sistema → aba "Hoje"
2. Preencher check-in (Fadiga · Dor · Sono · Motivação — escala 1 a 5)
3. Clicar em "Gerar Treino" (IA ou template automático)
4. Executar o treino
5. Clicar em "Registrar Conclusão"
6. Preencher os dados do treino (km, tempo, FC, zonas...)
7. Salvar
```

Se **Fadiga ≥ 4** ou **Dor ≥ 4**, o sistema ajusta automaticamente o tipo de treino:

| Planejado | Ajustado |
|---|---|
| Corrida Intensa | Corrida Leve |
| Corrida Moderada | Corrida Leve |
| Corrida Longa | Corrida Moderada |
| Tabata | Academia |

---

## Estrutura de Abas

### Hoje
- Tipo de treino do dia (calculado por data)
- Check-in de estado físico
- Geração de treino (IA ou fallback)
- Formulário de registro de conclusão

### Semana
- Grade visual dos 7 dias com status
- Métricas da semana: km total, treinos feitos, dias intensos, km do longo
- **Score semanal** com anel e 4 componentes
- Alertas automáticos
- Interpretação textual do score
- Plano detalhado da semana

### Dashboard
- 6 gráficos com histórico de até 16 semanas (Chart.js)
- Volume semanal
- Score semanal com cores por faixa
- Evolução da corrida longa
- Ritmo médio (eixo invertido — menor = mais rápido)
- FC média semanal
- Distribuição de zonas FC (doughnut da semana atual)

### Histórico
- Log de todos os treinos em ordem cronológica
- Distância, tempo, pace, FC, PSE e observações

### Config
- Chave API Anthropic
- Modo offline
- Referência de zonas de FC
- Exportar JSON / Apagar dados

---

## Registro de Treino — Campos

| Campo | Tipo | Obrigatório |
|---|---|---|
| Distância (km) | número | se corrida |
| Tempo (min) | número | se corrida |
| Pace médio | auto-calculado | — |
| FC média | número | recomendado |
| FC máxima | número | opcional |
| Zonas Z1–Z5 (%) | números (soma = 100) | recomendado |
| Fadiga (1–5) | número | sim |
| Dor muscular (1–5) | número | sim |
| PSE (1–10) | número | opcional |
| Observações | texto | opcional |

O **pace é calculado automaticamente** ao preencher km e tempo:
```
pace_seg = (tempo_min × 60) / km
```

---

## Sistema de Score (0 a 100)

Calculado ao final de cada semana com ao menos 3 treinos registrados.

```
score = (consistência × 0,30) + (intensidade × 0,20) + (recuperação × 0,20) + (eficiência × 0,30)
```

### Componentes

**Consistência** — treinos realizados vs planejados
- 100 pts = todos os treinos feitos
- −10 pts por treino perdido

**Intensidade** — dias com Z4+Z5 > 50%
- ≤ 2 dias → 100 pts
- 3 dias → 70 pts
- ≥ 4 dias → 40 pts

**Recuperação** — média de (fadiga + dor muscular) / 2
- ≤ 2,0 → 100 pts
- 2,1 – 3,0 → 70 pts
- ≥ 3,1 → 40 pts

**Eficiência** — comparação pace + FC vs semana anterior
- Pace melhorou E FC caiu → 100 pts
- Apenas um melhorou → 80 pts
- Nenhum melhorou → 55 pts
- Sem dados anteriores → 70 pts (neutro)

### Interpretação

| Score | Status | Ação |
|---|---|---|
| ≥ 85 | 🚀 Excelente | Pode progredir a carga |
| 70–84 | ✅ Bom | Manter estratégia |
| 50–69 | ⚠️ Atenção | Ajustar intensidade |
| < 50 | 🔴 Risco | Reduzir carga |

---

## Alertas Automáticos

| # | Condição | Alerta |
|---|---|---|
| 1 | Z4+Z5 > 60% em um treino | Treino muito intenso — reduzir carga |
| 2 | Mais de 2 dias intensos na semana | Excesso de intensidade semanal |
| 3 | Fadiga ≥ 4 em qualquer dia | Risco de sobrecarga |
| 4 | Volume > 20% acima da semana anterior | Aumento muito rápido de km |

---

## Estrutura Semanal Padrão

| Dia | Treino | Zona |
|---|---|---|
| Segunda | Corrida Leve | Z2 |
| Terça | Academia (Inferior/Superior alternado) | — |
| Quarta | Corrida Intensa (intervalado) | Z4–Z5 |
| Quinta | Tabata / Academia | Z4–Z5 |
| Sexta | Corrida Leve (sem. 1–8) / Moderada (sem. 9+) | Z2–Z3 |
| Sábado | Corrida Longa | Z2 |
| Domingo | Descanso Total | — |

### Semanas de Recuperação

A cada 4ª semana (4, 8, 12, 16...) o sistema entra em modo de recuperação:
- Corrida Intensa → Corrida Moderada
- Volume da corrida longa reduzido

---

## Progressão da Corrida Longa

O sistema calcula automaticamente a distância do longo por semana:

| Ciclo (4 sem.) | Sem. 1 | Sem. 2 | Sem. 3 | Sem. 4 (rec.) |
|---|---|---|---|---|
| 1 | 8 km | 9 km | 10 km | 8 km |
| 2 | 10 km | 11 km | 12 km | 10 km |
| 3 | 12 km | 13 km | 14 km | 12 km |
| 4 | 14 km | 15 km | 16 km | 14 km |
| 5 | 16 km | 17 km | 18 km | 16 km |
| 6 | 18 km | 19 km | 20 km | 18 km |
| 7 | 20 km | **21 km** 🎯 | — | — |

Meta de 21 km atingida na **semana 26** (~novembro 2026), com folga de ~5 semanas antes do objetivo de 27/12/2026.

---

## Zonas de FC

| Zona | Nome | FC |
|---|---|---|
| Z1 | Aquecimento | 86–102 bpm |
| Z2 | Fácil / Base aeróbica | 103–119 bpm |
| Z3 | Aeróbico constante | 120–136 bpm |
| Z4 | Limiar | 137–153 bpm |
| Z5 | Máximo | > 153 bpm |

---

## Regras do Sistema

1. Máximo de 2 treinos intensos por semana
2. Prioridade para treinos Z2 (base aeróbica)
3. Progressão da corrida longa: +1–2 km por semana
4. A cada 4 semanas: semana de recuperação automática
5. Domingo sempre descanso total
6. Tabata = conta como treino intenso da semana
7. Academia não pode prejudicar a corrida (cargas moderadas)
8. Fadiga ≥ 4 → intensidade reduzida automaticamente

---

## IA — Como Funciona

Com a chave API configurada, o sistema envia para o Claude:
- Dia da semana e semana do plano
- Tipo de treino planejado e km do longo
- Scores do check-in do dia
- Contexto do perfil e regras

O Claude retorna um treino estruturado com: nome, intensidade, zonas, duração, distância, ritmo, FC alvo, descrição em bullets, observação personalizada e frase motivacional.

**Sem chave API:** o sistema usa templates pré-configurados por tipo de treino e semana, igualmente detalhados.

---

## Dados e Privacidade

Todos os dados são armazenados **localmente** no `localStorage` do navegador. Nenhuma informação é enviada a servidores externos, exceto as chamadas à API Anthropic quando a chave está configurada.

### Exportar dados

Aba Config → **"Exportar JSON"** gera um arquivo `run50plus-YYYY-MM-DD.json` com todos os registros.

### Formato de exportação

```json
{
  "version": "run50plus",
  "exportedAt": "2026-05-04T10:00:00.000Z",
  "logs": {
    "2026-05-04": {
      "checkin": { "fadiga": 2, "dor": 1, "sono": 4, "motivacao": 5 },
      "adjustedType": "corrida_leve",
      "done": true,
      "metrics": {
        "km": 7.5,
        "time_min": 52.5,
        "pace": "7:00",
        "pace_sec": 420,
        "fc": 128,
        "fc_max": 144,
        "zonas_fc": { "z1": 5, "z2": 70, "z3": 20, "z4": 5, "z5": 0 },
        "pse": 5,
        "obs": "Treino tranquilo, bom ritmo."
      }
    }
  }
}
```

---

## Dependências

| Biblioteca | Versão | Uso | CDN |
|---|---|---|---|
| Chart.js | 4.4.1 | Gráficos | cdnjs.cloudflare.com |
| Google Fonts | — | Barlow + JetBrains Mono | fonts.googleapis.com |
| Anthropic API | 2023-06-01 | Geração de treino (opcional) | api.anthropic.com |

O sistema funciona offline (sem IA e sem gráficos) se as CDNs não estiverem acessíveis, exceto os gráficos que requerem Chart.js.

---

## Arquivos

```
run50plus.html     Sistema completo (single-file)
README.md          Esta documentação
```

---

*Run50+ · Luperce · Início do plano: 04/05/2026*
