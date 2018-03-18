# Elasticsearch



## 配置安装

### 安装es服务端

下载解压：https://www.elastic.co/downloads/elasticsearch

开启服务：

```bash
# 执行服务文件
/usr/local/var/elasticsearch/bin/elasticsearch
# 后台运行
./elasticsearch -d -p '/tmp/es_pid.log'
# 关闭服务
kill -9 `cat /tmp/es_pid.log`
```

访问：http://127.0.0.1:9200/，显示json信息即为安装成功。

### 安装head插件

```bash
# 下载
git clone https://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head && npm install
```

配置nginx站点，指向`/usr/local/var/elasticsearch/plugins/elasticsearch-head`目录。

支持跨域，修改es配置文件`config/elasticsearch.yml`：

```ini
# 添加2行配置
http.cors.enabled: true
http.cors.allow-origin: "*"
```

### 安装中文分词插件`IK`

```bash
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.2.2/elasticsearch-analysis-ik-6.2.2.zip
```

重启es服务

分词器：

ik_smart: 常规（粗粒度）拆分。

ik_max_word: 细粒度拆分，会穷尽所有可能，常用。

安装中分分词之后，需要给字段设置分词器：

```php
// 给main字段设置ik_smart分词器
$client->indices()->putMapping([
    'index' => 'index',
    'type'  => 'fulltext',
    'body'  => [
        'properties' => [
            'main' => [
                'type'            => 'text',
                'analyzer'        => 'ik_smart',
                'search_analyzer' => 'ik_smart',
            ],
        ],
    ],
]);
```

分词测试：

```php
# 如果测试失败，再在上index参数
$ret = $client->indices()->analyze([
    'body'     => [
        'text'      => '中华人民共和国国歌',
        'tokenizer' => 'ik_max_word',
    ],
]);
```

### 安装php api

```json
// composer.json 添加配置
{
    "require": {
    	"elasticsearch/elasticsearch": "~6.0"
    }
}
```

### 单机多节点

一个机器开启多个es节点的方法：复制整个es目录，再启动就行。

注意，如果提示node错误，注意清空新目录下的data文件夹。



## 概念释义

1. `index` 索引（名词），数据最顶层。
2. `type` 类型，数据第二层。
3. `id` 文档标识，一个文档对应一个id。
4. `document` 文档，一条数据即为一个文档。

路径：/megacorp/employee/1 即为索引 `megacorp` 下的 `employee` 类型的id为1的文档。



## 基础用法

### index 

> 索引（动词）文档，与名词的`索引`区分，即为添加一个文档索引，添加一条数据。

```php
$ret = $client->index([
    'index' => 'megacorp',
    'type'  => 'employee',
    'id'    => 3,
    'body'  => [
        'first_name' => 'Douglas',
        'last_name'  => 'Fir',
        'age'        => 35,
        'about'      => 'I like to build cabinets',
        'interests'  => ['forestry'],
    ],
]);
```

> **注意**：如果重发索引同一条数据（index,type,id相同），则会覆盖原数据，实现整条数据的更新。

### get

> 获取数据

```php
$ret = $client->get([
    'index' => 'megacorp',
    'type'  => 'employee',
    'id'    => 1,
]);
```

### delete

> 删除

```php
$ret = $client->get([
    'index' => 'megacorp',
    'type'  => 'employee',
    'id'    => 1,
]);
```

### update

局部更新

```php
// 更新main字段
$client->update([
    'index' => 'index',
    'type'  => 'fulltext',
    'id'    => 1,
    'body'  => [
        'doc' => [
            'main' => 123,
        ],
    ],
]);
```



## 搜索

### 轻量搜索

```php
# 搜索last_name为smith的文档
$ret = $client->search([
    'index' => 'megacorp',
    'type'  => 'employee',
    'q'     => ['last_name:smith'],
]);
print_r($ret);
```

### 相关性

在搜索`rock climbing`词组时，可能返回两个文档，一个匹配`rock climbing`另一个`rock albums`，返回结果里面有`_score`字段，即为相关性得分。

`相关性得分`：es会查询跟目标有相关性的文档，但不一定完全和需要查询的文字一致，所以会有相关性分值表示文档和查询的关键词的相似度。

### 查询类型

1. match 全文搜索，返回有相关性的文档。
2. math_phrase 短语搜索，精确匹配搜索短语。

### 高亮搜索

`highlight`参数：

```php
// 高亮about字段
'body' => [
    'highlight' => [
        'fields' => [
            'about' => new \stdClass(), // 空对象占位以使用默认高亮样式
        ]
    ]
]
```

