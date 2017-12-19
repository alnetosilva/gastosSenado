# Exemplo de calculo da media de gastos do senado
- Fazer uma consulta contendo por ano/mês, o valor médio de gastos do mês (independente de senador) e somente os senadores que gastaram mais que a média naquele mês específico.

### primeiro importe os dados para uma coleção temporária
```
mongoimport -d senado -c temp --file senado2017.json
```

### entre no mongo
$mongo senado

### Agora salve a colection temporaria em uma variavel par padronizarmos a data e os valores R$
```javascript
let base = db.temp.find()
```

### Agora mande executar um map para criarmos a nova coleção padronizada
```javascript
//executa o map no resultado do find
base.map(el => {
	//transforma o valor reembolsado em Number
	el.VALOR_REEMBOLSADO = Number(el.VALOR_REEMBOLSADO.replace(",","."));
	//atribui uma função de padronização à DATA
 	el.DATA = (d => {
		//padroniza a data no formato mm/dd/yyyy para transformarmos em date
 		return new Date(d[1]+"/"+d[0]+"/"+d[2]);
	 })
	 //auto executa a função passando como parametro de entrada um array contando a data
	 (el.DATA.split("/"));
	//insere o elemento padronizado no banco
 	db.reembolsos.insert(el)
})
```
### O que eu achei simplismente foooooddaaaaa foi a quantidade de reembolsos que esses caras pedem
```javascript
//Total reembolsado
db.reembolsos.aggregate(
	{$group:{
		_id:false,
		//Numero de pedidos de reembolso
		reembolsos:{$sum:1},
		//Soma dos reembolsos
		totalReembolsado:{$sum:"$VALOR_REEMBOLSADO"},
		//media de valor reembolsado
		mediaDeReembolso:{$avg:"$VALOR_REEMBOLSADO"},
		//menor valor reembolsado
		menorValor:{$min:"$VALOR_REEMBOLSADO"},
		//maior valor reembolsado
		maiorValor:{$max:"$VALOR_REEMBOLSADO"}
	}}
)
```
#### Resultado
```javascript
{ "_id" : false, "reembolsos" : 21291, "totalReembolsado" : 19878389.95, "mediaDeReembolso" : 933.6522450800808, "menorValor" : -243.4, "maiorValor" : 45000 }
```
### Somente um reembolso no periodo de amostragem foi devolvido
```javascript
db.reembolsos.find({VALOR_REEMBOLSADO:{$lte:0}}).pretty()
```
#### Resultado
```javascript
{
	"_id" : ObjectId("5a3866c409b487947506239d"),
	"ANO" : "2017",
	"MES" : "5",
	"SENADOR" : "DÁRIO BERGER",
	"TIPO_DESPESA" : "Contratação de consultorias, assessorias, pesquisas, trabalhos técnicos e outros serviços de apoio ao exercício do mandato parlamentar",
	"CNPJ_CPF" : "73.228.876/0001-63",
	"FORNECEDOR" : "TV CLIPAGEM LTDA. EPP.",
	"DOCUMENTO" : "001661",
	"DATA" : ISODate("2017-06-02T03:00:00Z"),
	"DETALHAMENTO" : "Despesa com Monitoramento de Informação Jornalística em Mídia Eletrônica e Imprensa",
	"VALOR_REEMBOLSADO" : -243.4
}

```
### Observem que os reembolsos são muito altos por mês. Mas essa base tá duvidosa também, a sequência de anos não bate
```javascript
//reembolsos por mês
db.reembolsos.aggregate(
	{$group:{
		_id:{ano:{$year:"$DATA"}, mes:{$month:"$DATA"}},
		//Soma dos reembolsos
		totalReembolsado:{$sum:"$VALOR_REEMBOLSADO"},
		//media de valor reembolsado
		mediaDeReembolso:{$avg:"$VALOR_REEMBOLSADO"},
		//menor valor reembolsado
		menorValor:{$min:"$VALOR_REEMBOLSADO"},
		//maior valor reembolsado
		maiorValor:{$max:"$VALOR_REEMBOLSADO"}
	}},
	//ordena por ano/mes
	{$sort:{"_id.ano":1,"_id.mes":1}}
)
```
#### Resultado
```javascript
{ "_id" : { "ano" : 201, "mes" : 3 }, "totalReembolsado" : 731.79, "mediaDeReembolso" : 731.79, "menorValor" : 731.79, "maiorValor" : 731.79 }
{ "_id" : { "ano" : 217, "mes" : 2 }, "totalReembolsado" : 430.51, "mediaDeReembolso" : 430.51, "menorValor" : 430.51, "maiorValor" : 430.51 }
{ "_id" : { "ano" : 2002, "mes" : 1 }, "totalReembolsado" : 471.42, "mediaDeReembolso" : 471.42, "menorValor" : 471.42, "maiorValor" : 471.42 }
{ "_id" : { "ano" : 2012, "mes" : 3 }, "totalReembolsado" : 1822, "mediaDeReembolso" : 1822, "menorValor" : 1822, "maiorValor" : 1822 }
{ "_id" : { "ano" : 2013, "mes" : 3 }, "totalReembolsado" : 1195.28, "mediaDeReembolso" : 1195.28, "menorValor" : 1195.28, "maiorValor" : 1195.28 }
{ "_id" : { "ano" : 2014, "mes" : 4 }, "totalReembolsado" : 16.41, "mediaDeReembolso" : 16.41, "menorValor" : 16.41, "maiorValor" : 16.41 }
{ "_id" : { "ano" : 2014, "mes" : 6 }, "totalReembolsado" : 399.15, "mediaDeReembolso" : 399.15, "menorValor" : 399.15, "maiorValor" : 399.15 }
{ "_id" : { "ano" : 2015, "mes" : 5 }, "totalReembolsado" : 140, "mediaDeReembolso" : 140, "menorValor" : 140, "maiorValor" : 140 }
{ "_id" : { "ano" : 2016, "mes" : 1 }, "totalReembolsado" : 976.9399999999999, "mediaDeReembolso" : 325.64666666666665, "menorValor" : 61.28, "maiorValor" : 615.66 }
{ "_id" : { "ano" : 2016, "mes" : 2 }, "totalReembolsado" : 3687.18, "mediaDeReembolso" : 1843.59, "menorValor" : 913.18, "maiorValor" : 2774 }
{ "_id" : { "ano" : 2016, "mes" : 3 }, "totalReembolsado" : 1657.77, "mediaDeReembolso" : 1657.77, "menorValor" : 1657.77, "maiorValor" : 1657.77 }
{ "_id" : { "ano" : 2016, "mes" : 4 }, "totalReembolsado" : 51.3, "mediaDeReembolso" : 51.3, "menorValor" : 51.3, "maiorValor" : 51.3 }
{ "_id" : { "ano" : 2016, "mes" : 5 }, "totalReembolsado" : 5913.78, "mediaDeReembolso" : 2956.89, "menorValor" : 1063.78, "maiorValor" : 4850 }
{ "_id" : { "ano" : 2016, "mes" : 6 }, "totalReembolsado" : 2245.75, "mediaDeReembolso" : 748.5833333333334, "menorValor" : 211.94, "maiorValor" : 1659.41 }
{ "_id" : { "ano" : 2016, "mes" : 7 }, "totalReembolsado" : 2941.65, "mediaDeReembolso" : 1470.825, "menorValor" : 950, "maiorValor" : 1991.65 }
{ "_id" : { "ano" : 2016, "mes" : 8 }, "totalReembolsado" : 180, "mediaDeReembolso" : 180, "menorValor" : 180, "maiorValor" : 180 }
{ "_id" : { "ano" : 2016, "mes" : 11 }, "totalReembolsado" : 662.85, "mediaDeReembolso" : 331.425, "menorValor" : 256.06, "maiorValor" : 406.79 }
{ "_id" : { "ano" : 2016, "mes" : 12 }, "totalReembolsado" : 101148.68, "mediaDeReembolso" : 1189.9844705882351, "menorValor" : 60.1, "maiorValor" : 27655.07 }
{ "_id" : { "ano" : 2017, "mes" : 1 }, "totalReembolsado" : 1211726.98, "mediaDeReembolso" : 849.7384151472651, "menorValor" : 2.3, "maiorValor" : 30000 }
{ "_id" : { "ano" : 2017, "mes" : 2 }, "totalReembolsado" : 1867164.78, "mediaDeReembolso" : 961.4648712667354, "menorValor" : 0.01, "maiorValor" : 30000 }
{ "_id" : { "ano" : 2017, "mes" : 3 }, "totalReembolsado" : 2172068.31, "mediaDeReembolso" : 881.5212297077923, "menorValor" : 2.32, "maiorValor" : 30000 }
{ "_id" : { "ano" : 2017, "mes" : 4 }, "totalReembolsado" : 2076308.36, "mediaDeReembolso" : 939.9313535536443, "menorValor" : 5, "maiorValor" : 30000 }
{ "_id" : { "ano" : 2017, "mes" : 5 }, "totalReembolsado" : 2463411.95, "mediaDeReembolso" : 956.2934588509318, "menorValor" : 0.01, "maiorValor" : 45000 }
{ "_id" : { "ano" : 2017, "mes" : 6 }, "totalReembolsado" : 2145873.85, "mediaDeReembolso" : 910.0398006785412, "menorValor" : -243.4, "maiorValor" : 31950 }
{ "_id" : { "ano" : 2017, "mes" : 7 }, "totalReembolsado" : 2018972.79, "mediaDeReembolso" : 976.7647750362845, "menorValor" : 0.21, "maiorValor" : 38500.96 }
{ "_id" : { "ano" : 2017, "mes" : 8 }, "totalReembolsado" : 2358835.79, "mediaDeReembolso" : 906.1989204763735, "menorValor" : 0.04, "maiorValor" : 38500.96 }
{ "_id" : { "ano" : 2017, "mes" : 9 }, "totalReembolsado" : 1986986.98, "mediaDeReembolso" : 908.5445724737083, "menorValor" : 0.01, "maiorValor" : 41000 }
{ "_id" : { "ano" : 2017, "mes" : 10 }, "totalReembolsado" : 1385993.58, "mediaDeReembolso" : 1051.5884522003034, "menorValor" : 6.25, "maiorValor" : 38500.96 }
{ "_id" : { "ano" : 2017, "mes" : 11 }, "totalReembolsado" : 61759.09, "mediaDeReembolso" : 2375.3496153846154, "menorValor" : 60, "maiorValor" : 10000 }
{ "_id" : { "ano" : 2017, "mes" : 12 }, "totalReembolsado" : 3397.44, "mediaDeReembolso" : 1132.48, "menorValor" : 211.19, "maiorValor" : 2900 }
{ "_id" : { "ano" : 2018, "mes" : 2 }, "totalReembolsado" : 32.8, "mediaDeReembolso" : 32.8, "menorValor" : 32.8, "maiorValor" : 32.8 }
{ "_id" : { "ano" : 2107, "mes" : 8 }, "totalReembolsado" : 1184.79, "mediaDeReembolso" : 1184.79, "menorValor" : 1184.79, "maiorValor" : 1184.79 }
```
### Reembolsos dos gastoes por mês

```javascript
db.reembolsos.aggregate(
	//Retornar a media mensal de reembolsos do Senado
	{$group:{
		//vamos agrupar as medias por mes e ano
		_id:{ano:{$year:"$DATA"}, mes:{$month:"$DATA"}},
		//calcula a media de valor reembolsado
		avgany:{$avg:"$VALOR_REEMBOLSADO"},
		//Soma dos reembolsos
		totalReembolsado:{$sum:"$VALOR_REEMBOLSADO"},
		//insere em uma subcolection os reembolsos correspondentes a esta media
		reembolsos:{$push:"$$ROOT"}}
	},
	//Desmembra a subcolection de reembolsos
	{$unwind:"$reembolsos"},
	//calcula o valor reembolsado por senador
	{$group:{
		//Agrupa os senadores para que possamos calcular os gastos deles por mes e ano, e mantenh a media mensal par amostragem
		_id:{gastao:"$reembolsos.SENADOR",ano:"$_id.ano",mes:"$_id.mes",avgMesGeral:"$avgany", sumMesGeral:"$totalReembolsado"},
		//calcula o valor reembolsado por mês do safado
		reembolsoSenadorMes:{$sum:"$reembolsos.VALOR_REEMBOLSADO"}
	}},
	//Retorna preenchido somente quem gastou mais que a media
	{$project:{
		//No id só vai vir quem estiver com valor maior que media mensal
		_id:{$cond:{
			if:{$gt:["$reembolsoSenadorMes","$_id.avgMesGeral"]},
			then:"$_id",
			else:false
		}},
		//a mesma coisa para o reembolso
		reembolsoSenadorMes:{$cond:{
			if:{$gt:["$reembolsoSenadorMes","$_id.avgMesGeral"]},
			then:"$reembolsoSenadorMes",
			else:false
		}}
	}},
	//agora só fica o que passou da media
	{$match:{
		_id:{$ne:false}
	}},
	//Limita em 3 por que a lista ficou grande
	{$limit:3}
)
```

#### Resultado
```javascript
{ "_id" : { "gastao" : "LÚCIA VÂNIA", "ano" : 2017, "mes" : 12, "avgMesGeral" : 1132.48, "sumMesGeral" : 3397.44 }, "reembolsoSenadorMes" : 2900 }
{ "_id" : { "gastao" : "GLADSON CAMELI", "ano" : 2016, "mes" : 6, "avgMesGeral" : 748.5833333333334, "sumMesGeral" : 2245.75 }, "reembolsoSenadorMes" : 1659.41 }
{ "_id" : { "gastao" : "ELMANO FÉRRER", "ano" : 2016, "mes" : 2, "avgMesGeral" : 1843.59, "sumMesGeral" : 3687.18 }, "reembolsoSenadorMes" : 2774 }

```
