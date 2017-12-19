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

### Corre pro abraço

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
