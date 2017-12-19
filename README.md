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
### Somente um reembolso no periodo de amostragem foi devolvido
```javascript
db.reembolsos.find({VALOR_REEMBOLSADO:{$lte:0}}).pretty()
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
	{$sort:{"_id.ano":1,"_id.mes":1}}
)
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
		_id:{gastao:"$reembolsos.SENADOR",ano:"$_id.ano",mes:"$_id.mes",mediaMensal:"$avgany", reembolsadoMes:"$totalReembolsado"},
		//calcula o valor reembolsado por mês do safado
		reembolso:{$sum:"$reembolsos.VALOR_REEMBOLSADO"}
	}},
	//Retorna preenchido somente quem gastou mais que a media
	{$project:{
		//No id só vai vir quem estiver com valor maior que media mensal
		_id:{$cond:{
			if:{$gt:["$reembolso","$_id.mediaMensal"]},
			then:"$_id",
			else:false
		}},
		//a mesma coisa para o reembolso
		reembolso:{$cond:{
			if:{$gt:["$reembolso","$_id.mediaMensal"]},
			then:"$reembolso",
			else:false
		}}
	}}
)
```
