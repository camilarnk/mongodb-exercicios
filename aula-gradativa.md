# Exercícios — Aula Gradativa — SÓ StreamTech 

BASE ÚNICA: stream.js — SÓ titulos, assinantes, visualizacoes 

## NÍVEL 1 — $match simples (aquecimento) 
### Ex 1 — $match simples 
Liste títulos do tipo 'filme'. Mostre titulo, tipo, notaImdb. Ordene por titulo.  
Resposta: 
```js
db.titulos.aggregate([
  {$match: {tipo: "filme"}},
  {$sort: {titulo: 1}},
  {$project: {_id: 0, titulo: 1, tipo: 1, notaImdb: 1}}
])
```
  
---

## NÍVEL 2 — $expr campo vs campo (sem conta) 
### Ex 2 — $expr + $size 
Quais títulos têm mais gêneros que pessoas no elenco? Mostre titulo, qtdGeneros, qtdElenco. 
Resposta: 

```js
db.titulos.aggregate([
  {$match: {$expr: {$gt: [{$size: "$generos"}, {$size: "$elenco"}]}}},
  {$project: {_id: 0, titulo: 1, qtdGeneros: {$size: "$generos"}, qtdElenco: {$size: "$elenco"}}}
])
```
   
### Ex 3 — $expr campo vs campo 
Quais títulos têm duracaoMin maior que notaImdb * 10? Mostre titulo, duracaoMin, notaImdb. (Aqui precisa de $expr 
porque é campo vs conta, não valor fixo.) 
Resposta:

```js
db.titulos.aggregate([
  {$match: {$expr: {$gt: ["$duracaoMin", {$multiply: ["$notaImdb", 10]}]}}},
  {$project: {_id: 0, titulo: 1, duracaoMin: 1, notaImdb: 1}}
])
```

---

## NÍVEL 3 — $expr com conta (90%) 
### Ex 4 — Sessão concluída 
Sessão concluída = assistiu >=90% da duração. Quantas concluídas por dispositivo? Use $lookup SIMPLES + $unwind + 
$expr. Mostre dispositivo, concluidas. 
Resposta: 

```js
db.visualizacoes.aggregate([
  {$lookup: {
    from: "titulos",
    localField: "tituloId",
    foreignField: "_id",
    as: "t"
  }},
  {$unwind: "$t"},
  {$match: {$expr: {$gt: ["$minutosAssistidos", {$multiply: ["$t.duracaoMin", 0.9]}]}}},
  {$project: {_id: 0,
              dispositivo: 1,
              sessaoConcluida: {$expr: {$gt: ["$minutosAssistidos", {$multiply: ["$t.duracaoMin", 0.9]}]}}
             }
  }
])
```
---

## NÍVEL 4 — $expr com $cond 
### Ex 5 — Curto demais 
Título curto demais se filme <95 ou série <50. Liste titulo, tipo, duracaoMin. 
Resposta:

```js
db.titulos.aggregate([
  {$match: {
    $expr: {
      $lt: ["$duracaoMin",
            {$cond: {
              if: {$eq: ["$tipo", "filme"]}, then: 95, else: 50
            }}]
    	}
  	}
  },
  {$project: {_id: 0, titulo: 1, tipo: 1, duracaoMin: 1}}
])
```

---

## NÍVEL 5 — $lookup simples + $replaceRoot 
### Ex 6 — $lookup simples 
Para cada visualização, traga titulo e duracaoMin do título. Use $lookup SIMPLES (sem pipeline, sem $expr dentro). 
Mostre dispositivo, minutosAssistidos, titulo, duracaoMin. 
Resposta: 

### Ex 7 — $replaceRoot 
Faça o mesmo $lookup do Ex 6, mas use $replaceRoot para que o título vire o documento principal. Mostre titulo, tipo, 
notaImdb. 
Resposta: 

---

## NÍVEL 6 — $unionWith simples (só titulos) 
### Ex 8 — Catálogo único 
Empilhe filmes + séries em lista só. Mesmos campos: titulo, tipo, notaImdb. Ordene por notaImdb desc. NÃO invente 
campo. 
Resposta: 

```js
db.titulos.aggregate([
  {$match: {tipo: "filme"}},
  {$unionWith: {
    	coll: "titulos",
    	pipeline: [{$match: {tipo: "serie"}}]
    }
  },
  {$sort: {notaImdb: -1}},
  {$project: {_id: 0, titulo: 1, tipo: 1, notaImdb: 1}}
])
```

---

## NÍVEL 7 — $map 
### Ex 9 — $map nomes do elenco 
Para cada título, liste só os nomes do elenco em nomesElenco. Use $map. Mostre titulo, nomesElenco. 
Resposta: 

---

## NÍVEL 8 — Índices (1 exemplo) 
### Ex 10 — Índice em tituloId 
1) Rode db.visualizacoes.find({ tituloId: 9 }).explain('executionStats') e anote docsExamined. 2) Crie 
db.visualizacoes.createIndex({ tituloId: 1 }). 3) Rode o explain de novo. O que mudou? Por quê? 
Resposta:

---

## Caça-bugs (só 2, bem simples) 
### Bug 1 
db.titulos.aggregate([ { $match: { $expr: { $gt: [ { $size:'generos' }, { $size:'$elenco' } ] } } } ]) 
Diagnóstico:
### Bug 2 
db.visualizacoes.aggregate([ { $match: { $expr: { $gte: ['$minutosAssistidos', '$t.duracaoMin'] } } 
} ]) 
Diagnóstico:
