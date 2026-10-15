Modelo dos dados
titulos:        { _id, titulo, tipo: "filme|serie", generos: [..], ano, duracaoMin, notaImdb, pais,
                  elenco: [{ nome, papel: "principal|coadjuvante" }] }
assinantes:     { _id, nome, email, plano: "basico|padrao|premium", cidade, uf, dataCadastro, ativo }
visualizacoes:  { _id, assinanteId, tituloId, dataHora, minutosAssistidos,
                  dispositivo: "tv|celular|web|tablet", avaliacao (opcional) }


Questão 1 — Sessões longas na web  •  coleção: visualizacoes
Liste as 3 maiores sessões de visualização (minutosAssistidos) feitas no dispositivo "web", com pelo menos 60 minutos assistidos, a partir de 01/04/2026 (inclusive). Em caso de empate nos minutos, use _id crescente. Retorne apenas assinanteId, tituloId e minutosAssistidos (sem _id).
Resposta:
```cmd
 db.visualizacoes.aggregate([
  {$match: {dispositivo: "web",
            minutosAssistidos: {$gt: 60},
           	dataHora: {$gte: new Date("2026-04-01T00:00:00.000+00:00")}
           }},
  {$sort: {minutosAssistidos: -1, _id: 1}},
  {$limit: 3},
  {$project: {_id: 0, assinanteId: 1, tituloId: 1, minutosAssistidos: 1}}
])
```

Questão 2 — Uso por dispositivo  •  coleção: visualizacoes
Para cada dispositivo, calcule totalVisualizacoes, minutosTotais e mediaMinutos (arredondada para 1 casa decimal). Retorne o campo dispositivo (sem _id) e ordene por minutosTotais decrescente.
Resposta:

```cmd
db.visualizacoes.aggregate([
  {$group: {_id: "$dispositivo",
            totalVisualizacoes: {$sum: 1},
           	minutosTotais: {$sum: "$minutosAssistidos"},
            mediaMinutos: {$avg: "$minutosAssistidos"}
           }},
  {$sort: {minutosTotais: -1}},
  {$project: {_id: 0,
              dispositivo: "$_id",
             	totalVisualizacoes: 1,
              minutosTotais: 1,
              mediaMinutos: {$round: ["$mediaMinutos", 1]}
             }}
])
```

Questão 3 — Gêneros mais frequentes  •  coleção: titulos
Cada título possui um array de gêneros. Encontre os 3 gêneros com mais títulos, retornando genero, qtd e notaMedia (média de notaImdb, 2 casas). Desempate por genero em ordem alfabética.
Resposta:

```cmd
db.titulos.aggregate([
  {$unwind: "$generos"},
  {$group: {_id: "$generos", qtd: {$sum: 1}, notaMedia: {$avg: "$notaImdb"}}},
  {$sort: {qtd: -1, _id: 1}},
  {$limit: 3},
  {$project: {_id: 0, generos: "$_id", qtd: 1, notaMedia: {$round: ["$notaMedia", 2]}}}
])
```

Questão 4 — Consumo por plano  •  coleção: visualizacoes
Usando a coleção de assinantes, calcule para cada plano os minutosTotais assistidos e quantos assinantesDistintos assistiram algo. Retorne plano, minutosTotais e assinantesDistintos, ordenados por minutosTotais decrescente.
Resposta:

```cmd
 db.visualizacoes.aggregate([
  {$lookup: {
    from: "assinantes",
    localField: "assinanteId",
    foreignField: "_id",
    as: "assinantes"
  }},
  {$group: {_id: "$assinantes.plano",
            minutosTotais: {$sum: "$minutosAssistidos"},
           	assinantesDistintos: {$sum: "$assinanteId"}
           }},
   {$sort: {minutosTotais: -1}},
   {$unwind: "$_id"},
   {$project: {_id: 0, plano: "$_id", minutosTotais: 1, assinantesDistintos: 1}}
])
```

Questão 5 — Perfil de engajamento  •  coleção: visualizacoes
Classifique cada visualização: "Maratona" (120 min ou mais), "Normal" (de 45 a 119 min) e "Abandono" (abaixo de 45). Para cada categoria, retorne a qtd de visualizações e a soma de minutos, ordenado por qtd decrescente.
Resposta:


Questão 6 — Faixas de duração  •  coleção: titulos
Distribua os títulos nas faixas de duracaoMin: [0,90), [90,120), [120,140) e "140 ou mais" (use o bucket padrão com esse nome). Para cada faixa retorne qtd, notaMedia (2 casas) e o array titulos com os nomes.
Resposta:
 

Questão 7 — Evolução mensal  •  coleção: visualizacoes
Calcule o total de minutos assistidos por mês, no formato "AAAA-MM". Retorne os campos mes e minutos, em ordem cronológica.
Resposta:


Questão 8 — Elenco principal   •  coleção: titulos
Para cada título, extraia o array principais com os nomes dos atores de papel "principal" e o campo qtdPrincipais. Mantenha apenas títulos com 2 ou mais atores principais. Retorne titulo (sem _id), principais e qtdPrincipais, ordenado por qtdPrincipais decrescente e titulo crescente.
Resposta:


Questão 9 — Painel executivo (uma única consulta)  •  coleção: visualizacoes
Com um único aggregate em visualizacoes, retorne um documento com três chaves: (a) porDispositivo: quantidade de visualizações por dispositivo, ordenada por qtd decrescente e _id crescente; (b) top3Titulos: os 3 tituloId com mais minutos (campo minutos), desempate por _id crescente; (c) resumo: totalVisualizacoes, minutosTotais e mediaMinutos (1 casa decimal), sem _id.
Resposta:


 
Questão 10 — Ranking com acumulado  •  coleção: visualizacoes
Some os minutos por tituloId e gere um ranking decrescente (desempate por _id crescente) com os campos posicao (rank) e acumulado (soma corrida dos minutos até a linha atual). Retorne apenas os 5 primeiros, com tituloId, minutos, posicao e acumulado. (Requer MongoDB 5.0+.)
Resposta:
 

