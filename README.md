<h1 align="center">Avengers Tower</h1>

<img align="right" src="AvengersTower.png" alt="Torre dos Vingadores em pixel art" width="220">

Banco de dados em MongoDB para gerenciar hospedagens na Torre dos Vingadores.

Inspirado na atividade **Hotel Nebula**, o projeto adapta quartos, serviços e atendimento às necessidades dos personagens da Marvel: estrutura reforçada para o Hulk, manutenção de armadura para Tony Stark e refeições depois das missões.

### Tecnologias

![MongoDB Atlas](https://img.shields.io/badge/MongoDB_Atlas-00684A?style=flat&logo=mongodb&logoColor=white)
![MongoDB Compass](https://img.shields.io/badge/MongoDB_Compass-115E59?style=flat&logo=mongodb&logoColor=white)
![JSON](https://img.shields.io/badge/JSON-333333?style=flat&logo=json&logoColor=white)
![VS Code](https://img.shields.io/badge/VS_Code-007ACC?style=flat)

**Situação:** dados importados no Compass e primeiras consultas realizadas. Análises e documentação em desenvolvimento.

<br clear="all">

<h2 align="center">Parte 1 — Planejamento</h2>

### 1. Modelagem

O banco `AvengersTower` possui oito coleções:

| Coleção | Responsabilidade |
| --- | --- |
| [hospedes](dados/hospedes.json) | Cadastro dos personagens, endereços e preferências. |
| [quartos](dados/quartos.json) | Acomodações, características e preços das diárias. |
| [reservas](dados/reservas.json) | Agendamento de quartos para os hóspedes. |
| [hospedagens](dados/hospedagens.json) | Entrada, saída e consumo durante a estadia. |
| [pagamentos](dados/pagamentos.json) | Valores recebidos, pagadores e formas de pagamento. |
| [funcionarios](dados/funcionarios.json) | Equipe, funções e permissões. |
| [servicos](dados/servicos.json) | Catálogo de alimentação, manutenção e atendimento. |
| [feedbacks](dados/feedbacks.json) | Avaliações dos hóspedes e respostas da equipe. |

### 1.2. Mapa de relações

```text
hospedes ─── reservas ─── quartos
                 │
             hospedagens ─── funcionarios
                 │
                 ├── servicos consumidos ─── servicos
                 ├── pagamentos
                 └── feedbacks
```

O mapa resume os principais vínculos. A tabela abaixo detalha
as referências armazenadas em cada coleção.

| Coleção | Referências | Dados incorporados |
| --- | --- | --- |
| `hospedes` | — | Endereço e preferências. |
| `quartos` | — | Características, comodidades e fotos. |
| `reservas` | Hóspede e quarto. | Cópias resumidas do hóspede e do quarto. |
| `hospedagens` | Reserva, hóspede, quarto e funcionários. | Serviços consumidos, com referência ao catálogo. |
| `pagamentos` | Hospedagem, reserva e hóspede. | Pagador e partes do pagamento. |
| `funcionarios` | — | Turno e permissões. |
| `servicos` | — | Horário disponível, tags e referência temática. |
| `feedbacks` | Hóspede, hospedagem e quarto. | Notas e resposta com identificação do funcionário. |

### 1.3. Decisões de modelagem

| Escolha | Motivo |
| --- | --- |
| Separar reserva e hospedagem. | A reserva planeja a estadia; a hospedagem registra o que aconteceu. |
| Incorporar endereço e preferências no hóspede. | São informações consultadas junto do cadastro. |
| Guardar cópias resumidas na reserva. | Preservam informações da ocasião, como a diária contratada. |
| Incorporar o consumo na hospedagem. | Mantém os itens e preços cobrados junto da conta. |
| Separar o catálogo de serviços. | Permite reutilizar serviços em diferentes hospedagens. |
| Manter feedbacks em coleção própria. | Permite várias avaliações por quarto sem aumentar seu documento. |

<h2 align="center">Parte 2 — Implementação</h2>

### 2. Organização dos arquivos

```text
AvengersTower/
├── README.md
├── assets/
│   └── AvengersTower.png
└── dados/
    ├── hospedes.json
    ├── quartos.json
    ├── reservas.json
    ├── hospedagens.json
    ├── pagamentos.json
    ├── funcionarios.json
    ├── servicos.json
    └── feedbacks.json
```

### 2.1. Como executar

A implementação atual utiliza arquivos JSON importados pelo Compass.

| Etapa | Ação |
| --- | --- |
| 1. Conectar | Abra no Compass a conexão com o MongoDB Atlas. |
| 2. Criar o banco | Crie o banco `AvengersTower`. |
| 3. Criar as coleções | Use os nomes dos arquivos, sem a extensão `.json`. |
| 4. Importar | Em cada coleção, escolha **ADD DATA → Import JSON or CSV** e selecione o arquivo correspondente. |
| 5. Conferir | Abra os documentos e execute as consultas da Parte 3. |

Cada objeto da lista JSON é importado como um documento independente.
Importe cada arquivo uma vez para evitar conflitos de `_id`.

### 2.2. Padrões dos dados

| Informação | Representação |
| --- | --- |
| Identificadores | Textos definidos manualmente, como `"h04"` e `"q02"`. |
| Referências | Campos como `hospede_id`, com o mesmo identificador do documento relacionado. |
| Datas | Extended JSON com `$date`, importado como data pelo MongoDB. |
| Valores monetários | Números expressos em dólares americanos (`USD`). |
| Diárias | Preço do quarto multiplicado pela quantidade de noites. |
| Total da hospedagem | Soma das diárias e dos serviços consumidos. |

<h2 align="center">Parte 3 — Consultas</h2>

### 3. Consultas no Compass

Na aba **Documents**, informe o filtro e clique em **Find**.

#### Quartos com estrutura reforçada

Coleção: `quartos`.

```json
{
  "caracteristicas.estrutura_reforcada": true
}
```

#### Quartos para duas pessoas com diária de até USD 500

Coleção: `quartos`.

```json
{
  "capacidade_max": { "$gte": 2 },
  "preco_diaria": { "$lte": 500 }
}
```

#### Reserva do Hulk

Coleção: `reservas`.

```json
{
  "hospede_id": "h04"
}
```

### 3.1. Análises a concluir

| Análise | Situação |
| --- | --- |
| Hóspedes com mais de uma reserva. | Agrupamento realizado; filtro final a revisar. |
| Receita recebida por período. | Pendente. |
| Períodos com mais reservas. | Pendente. |
| Reservas em outros meses. | Ampliação dos dados planejada. |

O conjunto inicial possui uma reserva por hóspede, todas em
novembro de 2026. Novas reservas permitirão comparar períodos
e identificar hóspedes recorrentes.

## Referência

Atividade [Hotel Nebula — Gabriel Augusto](https://github.com/gabaugusto/sample-databases/tree/main/Documentos/6_HOTEL_NEBULA).

Projeto acadêmico com dados fictícios e temática inspirada na Marvel.

## Contexto acadêmico

Os dados representam uma simulação para estudo. Serviços, preços, falas e adaptações dos personagens foram criados para a atividade e não representam um sistema oficial da Marvel. O material do professor fundamenta a proposta; o tema Avengers Tower personaliza o cenário.
