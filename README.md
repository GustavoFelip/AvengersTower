# Avengers Tower — MongoDB

Projeto de banco de dados NoSQL para hospedagens na Torre dos Vingadores, baseado na atividade [Hotel Nebula, do professor Gabriel Augusto](https://github.com/gabaugusto/sample-databases/tree/main/Documentos/6_HOTEL_NEBULA).

Os hóspedes são personagens da Marvel. Os quartos e serviços atendem às suas necessidades: estrutura reforçada para o Hulk, manutenção de armadura para Tony Stark e manutenção de lançadores de teia para Peter Parker.

**Status: em desenvolvimento.** Os dados iniciais foram importados no MongoDB Compass. As consultas de análise e a documentação final ainda serão concluídas.

## Tecnologias

MongoDB para armazenamento, MongoDB Compass para importação e consultas, VS Code para editar os arquivos e GitHub para versionamento e documentação.

## Coleções

| Coleção e arquivo | Documentos | Finalidade |
| --- | ---: | --- |
| [hospedes.json](hospedes.json) | 7 | Heróis, endereço e preferências. |
| [quartos.json](quartos.json) | 15 | Capacidade, diária e características. |
| [reservas.json](reservas.json) | 7 | Planejamento das estadias e vínculo entre hóspede e quarto. |
| [hospedagens.json](hospedagens.json) | 7 | Entrada, saída, serviços consumidos e totais. |
| [pagamentos.json](pagamentos.json) | 7 | Pagadores, valores e formas de pagamento. |
| [funcionarios.json](funcionarios.json) | 5 | Equipe responsável pela torre. |
| [servicos.json](servicos.json) | 19 | Alimentação, manutenção e atendimento. |
| [feedbacks.json](feedbacks.json) | 7 | Avaliações e respostas da torre. |

Cada arquivo contém uma lista de documentos. Ao importar, cada objeto se torna um documento independente na coleção correspondente.

## Modelagem

O banco se chama `AvengersTower`. Os documentos usam `_id` textuais definidos manualmente, como `h04` para o Hulk e `q02` para o quarto número 102. O identificador é único dentro da coleção.

### Referências

- Reservas ligam hóspedes e quartos por `hospede_id` e `quarto_id`.
- Hospedagens referenciam reserva, hóspede, quarto e funcionários de entrada e saída. Cada serviço consumido contém `servico_id`.
- Pagamentos referenciam hospedagem, reserva e hóspede.
- Feedbacks referenciam hóspede, hospedagem e quarto; a resposta identifica o funcionário.

Exemplo: Hulk (`h04`) → reserva (`r04`) → quarto (`q02`, número 102). Os códigos precisam coincidir em valor e tipo. Essas referências são mantidas nos dados; o MongoDB não as verifica automaticamente como chaves estrangeiras.

### Dados incorporados

Endereço e preferências ficam dentro do hóspede porque são consultados com ele. Características e comodidades ficam no quarto. Serviços consumidos ficam em uma lista na hospedagem, facilitando a consulta dos itens e do total da estadia.

As reservas guardam `hospede_snapshot` e `quarto_snapshot`: cópias de informações no momento da reserva, incluindo a diária. Isso preserva o contexto mesmo que o cadastro original mude. O catálogo de serviços é separado para reutilização; cada consumo guarda o preço aplicado naquela ocasião.

### Datas e valores

Datas usam MongoDB Extended JSON, por exemplo: `{ "$date": "2026-11-10T14:00:00Z" }`.

Os valores monetários são em dólares americanos (`USD`). Diárias dependem do quarto e da quantidade de noites; o total da hospedagem soma diárias e consumo. Alguns pagamentos são custeados por patrocinadores, como as Indústrias Stark.

## Importação no Compass

1. Conecte o Compass ao servidor MongoDB local ou Atlas.
2. Crie o banco `AvengersTower` e a coleção `hospedes`.
3. Escolha **ADD DATA → Import JSON or CSV** e importe `hospedes.json`.
4. Crie as outras coleções usando os nomes dos arquivos sem `.json` e importe cada arquivo na coleção correspondente.
5. Confira as quantidades da tabela e os campos de data.

Importe cada arquivo uma vez: repetir a importação pode gerar erros de `_id` duplicado. O nome da conexão é livre. No ambiente de estudo, `Atividade_Oscar` é a conexão e `AvengersTower` é o banco.

## Consultas praticadas

Na aba **Documents**, escolha a coleção, cole o filtro e clique em **Find**. **Reset** limpa o filtro.

| Coleção | Objetivo | Filtro |
| --- | --- | --- |
| hospedes | Localizar o Hulk | `{ "nome_heroi": "Hulk" }` |
| quartos | Estrutura reforçada | `{ "caracteristicas.estrutura_reforcada": true }` |
| quartos | Capacidade de pelo menos duas pessoas | `{ "capacidade_max": { "$gte": 2 } }` |
| quartos | Duas pessoas e diária até USD 500 | `{ "capacidade_max": { "$gte": 2 }, "preco_diaria": { "$lte": 500 } }` |
| reservas | Reserva do Hulk | `{ "hospede_id": "h04" }` |
| quartos | Quarto reservado pelo Hulk | `{ "_id": "q02" }` |

Essas consultas foram executadas durante o estudo no Compass. A busca por estrutura reforçada retornou três quartos. A consulta do quarto `q02` mostrou o número 102.

### Contagem de reservas por hóspede

Na aba **Aggregations** de `reservas`, o estágio `$group` foi executado para contar as reservas por hóspede. A etapa seguinte, `$match`, seleciona quem possui mais de uma reserva e será revisada na continuação do estudo.

Pipeline completo para usar no modo **TEXT**:

```json
[
  {
    "$group": {
      "_id": "$hospede_id",
      "total_reservas": { "$sum": 1 }
    }
  },
  {
    "$match": {
      "total_reservas": { "$gt": 1 }
    }
  }
]
```

Com os dados atuais, o resultado esperado do pipeline completo é vazio: cada hóspede tem uma reserva. Cada estágio recebe o resultado do anterior; `total_reservas` é calculado durante a consulta e não altera os documentos salvos.

## Próximas etapas

- [x] Definir o tema e a estrutura das oito coleções.
- [x] Criar os arquivos JSON e importar no Compass.
- [x] Praticar filtros por características e referências.
- [x] Executar o agrupamento das reservas por hóspede.
- [ ] Revisar e executar o filtro de hóspedes recorrentes após o agrupamento.
- [ ] Calcular receita recebida em um período.
- [ ] Identificar os períodos com mais reservas.
- [ ] Adicionar reservas em outros meses e repetir hóspedes.
- [ ] Registrar resultados finais e revisar a documentação.

As sete reservas iniciais estão em novembro de 2026. A ampliação dos dados fica para uma próxima etapa; comparações entre meses ainda são limitadas a esse conjunto.

## Contexto acadêmico

Os dados representam uma simulação para estudo. Serviços, preços, falas e adaptações dos personagens foram criados para a atividade e não representam um sistema oficial da Marvel. O material do professor fundamenta a proposta; o tema Avengers Tower personaliza o cenário.
