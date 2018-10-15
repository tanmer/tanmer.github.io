# select  jsonb

例:

```text
table_name  -> company_contents
column_name -> content
column_type -> jsonb
data        ->  {
                  "企业类型": '有限公司'，
                  "经营范围": 'IT',
                  "企业资质": {
                    "主板上市": true
                  }
                }

# 查询 ( 企业类型 = '有限公司' ) 的所有企业

select * from company_contens where ( content ->>  '企业类型' = '有限公司' );

# 查询 ( 主板上市 = true ) 的所有企业

select * from company_contents where ( content -> '企业资质' ->> '主板上市' = ture )
```

上面主要运用了 -&gt; 和 -&gt;&gt; 做查询，-&gt;  作用是获取 json 对象，-&gt;&gt;  则是获取一个文本信息，详细请查阅官方文档: [https://www.postgresql.org/docs/9.6/static/functions-json.html](https://www.postgresql.org/docs/9.6/static/functions-json.html)

