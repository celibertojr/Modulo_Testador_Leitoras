# 🃏 Módulo Testador de Leitoras de Cartões Magnéticos

> Equipamento automatizado para ensaio de confiabilidade de leitoras de cartão magnético, desenvolvido em parceria com a **Itautec**.  
> Firmware em Assembly para **PIC 16F877A** · Projeto de Formatura · FEI, 2003–2004

---

## 📋 Descrição

Leitoras de cartão magnético utilizadas em caixas eletrônicos possuem vida útil e parâmetros de desempenho especificados pelo fabricante — mas nem sempre condizentes com a realidade de campo. Este módulo foi criado para **testar automaticamente** essas leitoras antes de sua implantação, simulando os ciclos reais de inserção e remoção de cartão e contabilizando leituras bem-sucedidas e com erro.

O sistema é uma melhoria do equipamento existente na Itautec, com foco em:

- ✅ Confiabilidade das leituras
- ✅ Verificação de vida útil (contagem de ciclos)
- ✅ Validação de velocidade de leitura
- ✅ Registro de leituras erradas com exibição em tempo real

---

## ⚙️ Funcionamento

```
┌─────────────┐     RS-232      ┌─────────────────────┐
│  PIC 16F877A│◄───────────────►│  Leitora de Cartão  │
│  (firmware) │                 │  Magnético (DUT)    │
└──────┬──────┘                 └─────────────────────┘
       │ UCN5804                          ▲
       ▼                                  │
┌─────────────┐                  ┌────────┴────────┐
│ Motor de    │──── braço ──────►│    Cartão       │
│   Passo     │     mecânico     │   (inserção /   │
└─────────────┘                  │    remoção)     │
                                 └─────────────────┘
       │
       ▼
┌─────────────┐
│ LCD 2×16   │  Exibe: ciclos, erros, velocidade, status
│ 6 Botões   │  Configura: parâmetros do ensaio
│ 6 LEDs     │  Indica: estado do sistema
└─────────────┘
```

Um motor de passo, acionado pelo driver UCN5804, movimenta um braço mecânico que insere e remove o cartão na leitora de forma repetida e controlada. O microcontrolador gerencia todo o ciclo, se comunica com a leitora via RS-232 e exibe os resultados em um display LCD.

---

## 🔧 Hardware

| Componente | Descrição |
|---|---|
| **Microcontrolador** | PIC 16F877A — 8 bits, 8 KB Flash, 368 B RAM |
| **Driver de motor** | UCN5804 — motor de passo unipolar |
| **Display** | LCD 16×2, barramento 8 bits (PORTD) |
| **Interface serial** | RS-232 com handshake RTS/CTS (USART do PIC) |
| **Alimentação** | Transformador 110V/9V → regulador 5V |
| **IHM** | 6 botões (PORTB), 6 LEDs (PORTA/PORTE) |

### Pinagem principal (PIC 16F877A)

| PORT | Função |
|---|---|
| `PORTA,0–3,5` | LEDs 0–4 |
| `PORTB,0–5` | Botões BT0–BT5 |
| `PORTC,0` | RESET da leitora |
| `PORTC,1–3` | STEP_ENABLE / DIRECTION / STEP_INPUT (motor) |
| `PORTC,4–7` | RTS / CTS / TX / RX (serial RS-232) |
| `PORTD,0–7` | Barramento de dados LCD (8 bits) |
| `PORTE,0–1` | RS / ENABLE do LCD |
| `PORTE,2` | LED 5 |

---

## 💾 Firmware

Escrito inteiramente em **Assembly MPASM** para o PIC 16F877A.

### Estrutura do código (`PROJETO.asm`, 2.259 linhas)

```
ORG 0x0000  → Vetor de Reset  → GOTO CONFIG
ORG 0x0004  → ISR (Interrupções)
               ├── TMR2IF  → Pulso STEP_INPUT (motor de passo)
               └── RCIF    → Recepção serial (USART)

CONFIG      → Inicialização de ports, TMR2, USART, pull-ups
INICIALIZACAO_DISPLAY → Sequência de init do LCD
INIC_LEITORA          → Reset + handshake inicial com a leitora

MAIN        → Loop principal
               ├── Varredura de botões (com filtro anti-bounce)
               ├── TRATA_BT0..3 → ajuste de parâmetros
               └── START → inicia ciclo de ensaio

Ciclo de ensaio:
  INSERE → [aguarda] → LE_BUFFER_LEITORA → [OK/ERRO] → REMOVE → SHOW_LCD
```

### Protocolo serial com a leitora

| Byte | Significado |
|---|---|
| `0x02` | STX — início de mensagem |
| `0x03` | ETX — fim de mensagem (dispara processamento) |
| `0x06` | ACK — confirmação positiva |
| `'N'` (0x4E) | Leitura com erro |
| `'P'` (0x50) | Leitura OK |
| `'R'` (0x52) | Consulta de status (Revision) |

---

## 📂 Arquivos do Repositório

| Arquivo | Descrição |
|---|---|
| `PROJETO.asm` | Firmware principal em Assembly (MPASM) |
| `projetodeformatura.doc` | Relatório completo do projeto de formatura |
| `Fluxograma de Funcionamento.doc` | Fluxogramas do sistema e do programa |
| `Apresentacao.ppt` | Slides de apresentação |
| `Material.doc` | Lista de materiais com fornecedores (~R$2.400) |
| `Projeto.dwg` | Esquema elétrico (AutoCAD) |
| `sistema.dwg` | Diagrama mecânico do sistema (AutoCAD) |

---

## 🛠️ Como compilar o firmware

1. Instale o **MPLAB IDE** (Microchip) ou o **MPLAB X IDE**
2. Adicione o arquivo `PROJETO.asm` a um novo projeto com o device `PIC16F877A`
3. Inclua o arquivo de definições `P16F877A.INC` (incluso na instalação do MPLAB)
4. Compile (Build) — o assembler gerará o arquivo `.hex`
5. Grave o `.hex` no PIC usando um programador compatível (ICD2, PICkit, etc.)

> **Oscilador:** Configure o hardware para cristal HS. A CONFIG WORD já está definida no topo do `.asm`.

---

## 👥 Equipe

Projeto desenvolvido na **Faculdade de Engenharia Industrial — FEI**, São Bernardo do Campo, SP.

| Nome | RA |
|---|---|
| Renato Franhani | 10299362-3 |
| Marcus Vinicius Fuso Voltarelli | 10299425-8 |
| Fabrício Dias Pereira | 10299450-6 |
| Bianca Bizan | 10299456-3 |
| Luiz Antonio Celiberto Jr. | 10299469-6 |
| Rodrigo Andrade Felício | 10299480-3 |

**Orientador:** Prof. Ricardo Stolf

---

## 📚 Referências

- Microchip Technology. [*PIC16F877A Datasheet* (DS40001272)](https://www.microchip.com/en-us/product/PIC16F877A)
- Microchip Technology. [*MPASM Assembler User's Guide* (DS33014)](https://www.microchip.com)
- Allegro MicroSystems. [*UCN5804B Stepper Motor Driver Datasheet*](https://www.allegromicro.com)

---

<sub>Projeto de Formatura · FEI / Itautec · 2003–2004 · Publicado no GitHub em 2015</sub>
