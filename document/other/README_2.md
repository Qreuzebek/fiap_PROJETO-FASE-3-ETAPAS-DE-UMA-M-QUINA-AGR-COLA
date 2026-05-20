
# Importação e consulta de dados -  Oracle SQL Developer

## i. Gerando dados

Como os sensores da fase dois geraram poucos dados, decidi usar a ajuda do Claude para gerar dados factíveis:

- Os dados representam as leituras e deciões dos sensores
- Compreendem ao período de janeiro de 2024 à maio de 2026
- A períodicidade de coleta simulada foi de 1x / dia
- A base foi dividida em duas 
    - i. Leituras - KPIS dos sensores 
    - ii.Açoes Irrigação - Comando enviado ao rele

Os dados também estão prsentes em formato CSV no repositório. 

## ii. Importando dados no banco

### Importando as leituras dos sensores:

![alt text](image.png)

Dados importados com sucesso: 

![alt text](image-1.png)

Descrição da tabela: 

![alt text](image-2.png)

Dados da tabela (head 10):

![alt text](image-3.png)

### Importando dados das ações dos irridaroes: 

Descrição da tabela: 

![alt text](image-4.png)

Dados da tabela (head 10)
![alt text](image-5.png)

## iii. Consultando dados:

### Consulta i.: Lista dos dias que o irrigador foi acionado: 

##### Query: 
```sql
SELECT
    id,
    leitura_id,
    CAST(decidido_em AS DATE) AS data_irrigacao,
    decidido_em,
    codigo_acao,
    mensagem
FROM acoes_irrigador
WHERE rele_ativado = 'True'
ORDER BY decidido_em;
```

![alt text](image-6.png)

### Consulta ii. Quantas vezes cada motivo suspendeu o acionamento do irrigador: 

#### Query
 ```sql
 SELECT
    codigo_acao,
    mensagem,
    COUNT(*) AS total_suspensoes
FROM acoes_irrigador
WHERE rele_ativado = 'False'
GROUP BY codigo_acao, mensagem
ORDER BY total_suspensoes DESC;
 ```

![alt text](image-7.png)

### Consulta iii.: Será que o meu sensor está avisando corretamente quando a umidade está acima dos 80%?

#### Query

```sql
SELECT
    s.id              AS leitura_id,
    CAST(s.coletado_em AS DATE) AS data,
    s.umidade,
    a.codigo_acao,
    a.mensagem,
    CASE
        WHEN s.umidade > 80 AND a.mensagem LIKE '%Umidade acima de 80%'
            THEN 'OK'
        WHEN s.umidade > 80 AND a.mensagem NOT LIKE '%Umidade acima de 80%'
            THEN 'DIVERGENTE'
        ELSE
            'N/A'
    END AS consistencia
FROM leitura_sensores s
JOIN acoes_irrigador a ON a.leitura_id = s.id
ORDER BY data;
```

![alt text](image-8.png)

Podemos notar que em alguns casos há inconsistência, mas tudo bem, isso é esperado pois na lógica que criamos (consultar aqui) só o aviso mais relevante na orderm de hierárquia é exibido. 


## iv. Conclusão 

Podemos concluir que as bases foram importadas com sucesso no banco de dados e que as consultas estão funcionando como deveram. A chave ```s.id ``` funcionou adequadamente chave entre as duas bases. 


