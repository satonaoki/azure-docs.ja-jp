---
title: "新しいスキーマ バージョン 2015-08-01-preview"
description: "最新バージョンのロジック アプリの JSON 定義を記述する方法について説明します。"
author: stepsic-microsoft-com
manager: anneta
editor: 
services: logic-apps
documentationcenter: 
ms.assetid: 0d03a4d4-e8a8-4c81-aed5-bfd2a28c7f0c
ms.service: logic-apps
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/31/2016
ms.author: stepsic
translationtype: Human Translation
ms.sourcegitcommit: 061b68ec5fc9c14fd0c3f00cc0be9f09717fc1ad
ms.openlocfilehash: 8f1e2680aff5c1d5c9b90654a334cae2245db5fb
ms.lasthandoff: 02/15/2017


---
# <a name="new-schema-version-2015-08-01-preview"></a>新しいスキーマ バージョン 2015-08-01-preview
Azure Logic Apps の新しいスキーマと API バージョンでは、ロジック アプリの信頼性と使いやすさを向上させる複数の機能強化が行われています。 次の&4; つの主な違いがあります。

1. アクションの種類 **APIApp** が更新され、新しいアクションの種類 **APIConnection** になりました。
2. **Repeat** の名前が **Foreach** に変更されました。
3. **HTTP リスナー** API アプリが不要になりました。
4. 子ワークフローの呼び出しに新しいスキーマを使用します。

## <a name="1-moving-to-api-connections"></a>1.API 接続への移行
最大の変更は、API を使用するために、Azure サブスクリプションに API アプリをデプロイする必要がなくなった点です。 API は、次の 2 とおりの方法で使用できます。

* マネージ API
* カスタム Web API

これらの API は、管理モデルとホスティング モデルが異なることから、それぞれの処理方法も若干異なります。 このモデルのメリットの&1; つは、リソース グループにデプロイされたリソース以外にも、リソースを使用できるようになったことです。 

### <a name="managed-apis"></a>マネージ API
Microsoft では、Office 365、Salesforce、Twitter、FTP などの API の一部を代わりに管理しています。 このようなマネージ API には、Bing 翻訳のようにそのまま使用できる API もあれば、構成が必要な API もあります。 このような構成は "*接続*" と呼ばれています。

たとえば、Office 365 を使用するときは、Office 365 のサインイン トークンを含む接続を作成する必要があります。 このトークンは、ロジック アプリでいつでも Office 365 API を呼び出すことができるように、セキュリティで保護された状態で保存と更新が行われます。 また、SQL サーバーや FTP サーバーに接続する場合は、接続文字列を含む接続を作成する必要があります。 

これらのアクションは、定義では `APIConnection` と呼ばれます。 電子メールを送信するために Office 365 を呼び出す接続の例を次に示します。

```
{
    "actions": {
        "Send_Email": {
            "type": "ApiConnection",
            "inputs": {
                "host": {
                    "api": {
                        "runtimeUrl": "https://msmanaged-na.azure-apim.net/apim/office365"
                    },
                    "connection": {
                        "name": "@parameters('$connections')['shared_office365']['connectionId']"
                    }
                },
                "method": "post",
                "body": {
                    "Subject": "Reminder",
                    "Body": "Don't forget!",
                    "To": "me@contoso.com"
                },
                "path": "/Mail"
            }
        }
    }
}
```

`host` オブジェクトは、API 接続固有の入力部分であり、`api` と `connection` の&2; つのパートが含まれています。

`api` には、マネージ API がホストされる場所のランタイム URL が含まれています。 `GET https://management.azure.com/subscriptions/{subid}/providers/Microsoft.Web/managedApis/?api-version=2015-08-01-preview` を呼び出すと、利用可能なマネージ API すべてを表示することができます。

API を使用する場合、その API には**接続パラメーター**が定義されている場合と定義されていない場合があります。 定義されていない場合、**接続**は必要ありません。 定義されている場合は、接続を作成する必要があります。 接続を作成する際に、選択した名前が設定されます。 そして、`host` オブジェクト内の `connection` オブジェクトの名前を参照することになります。 リソース グループ内に接続を作成するには、次の行を呼び出します。

```
PUT https://management.azure.com/subscriptions/{subid}/resourceGroups/{rgname}/providers/Microsoft.Web/connections/{name}?api-version=2015-08-01-preview
```

さらに以下の本体を追加します。

```
{
  "properties": {
    "api": {
      "id": "/subscriptions/{subid}/providers/Microsoft.Web/managedApis/azureblob"
    },
    "parameterValues": {
        "accountName": "{The name of the storage account -- the set of parameters is different for each API}"
    }
  },
  "location": "{Logic app's location}"
}
```

### <a name="deploying-managed-apis-in-an-azure-resource-manager-template"></a>Azure Resource Manager テンプレートでマネージ API をデプロイする
作成するアプリケーションに対話型サインインが必要ない場合、そのアプリケーション全体を Azure Resource Manager テンプレートで作成できます。
対話型サインインが必要な場合は、Azure Resource Manager テンプレートですべて設定できますが、さらにポータルにアクセスして接続を承認する必要があります。 

```
    "resources": [{
        "apiVersion": "2015-08-01-preview",
        "name": "azureblob",
        "type": "Microsoft.Web/connections",
        "location": "[resourceGroup().location]",
        "properties": {
            "api": {
                "id": "[concat(subscription().id,'/providers/Microsoft.Web/locations/westus/managedApis/azureblob')]"
            },
            "parameterValues": {
                "accountName": "[parameters('storageAccountName')]",
                "accessKey": "[parameters('storageAccountKey')]"
            }
        }
    }, {
        "type": "Microsoft.Logic/workflows",
        "apiVersion": "2015-08-01-preview",
        "name": "[parameters('logicAppName')]",
        "location": "[resourceGroup().location]",
        "dependsOn": ["[resourceId('Microsoft.Web/connections', 'azureblob')]"
        ],
        "properties": {
            "sku": {
                "name": "[parameters('sku')]",
                "plan": {
                    "id": "[concat(resourceGroup().id, '/providers/Microsoft.Web/serverfarms/',parameters('svcPlanName'))]"
                }
            },
            "definition": {
                "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2015-08-01-preview/workflowdefinition.json#",
                "actions": {
                    "Create_file": {
                        "type": "apiconnection",
                        "inputs": {
                            "host": {
                                "api": {
                                    "runtimeUrl": "https://logic-apis-westus.azure-apim.net/apim/azureblob"
                                },
                                "connection": {
                                    "name": "@parameters('$connections')['azureblob']['connectionId']"
                                }
                            },
                            "method": "post",
                            "queries": {
                                "folderPath": "[concat('/', parameters('containerName'))]",
                                "name": "helloworld.txt"
                            },
                            "body": "@decodeDataUri('data:, Hello+world!')",
                            "path": "/datasets/default/files"
                        },
                        "conditions": []
                    }
                },
                "contentVersion": "1.0.0.0",
                "outputs": {},
                "parameters": {
                    "$connections": {
                        "defaultValue": {},
                        "type": "Object"
                    }
                },
                "triggers": {
                    "recurrence": {
                        "type": "Recurrence",
                        "recurrence": {
                            "frequency": "Day",
                            "interval": 1
                        }
                    }
                }
            },
            "parameters": {
                "$connections": {
                    "value": {
                        "azureblob": {
                            "connectionId": "[concat(resourceGroup().id,'/providers/Microsoft.Web/connections/azureblob')]",
                            "connectionName": "azureblob",
                            "id": "[concat(subscription().id,'/providers/Microsoft.Web/locations/westus/managedApis/azureblob')]"
                        }

                    }
                }
            }
        }
    }]
```

この例では、接続がリソース グループ内に存在する単なるリソースであることがわかります。 これらの接続は、ユーザーのサブスクリプション内で利用可能なマネージ API を参照しています。

### <a name="your-custom-web-apis"></a>カスタム Web API
独自の API (特に、Microsoft によって管理されていない API) を使用している場合、それを呼び出すには組み込みの **HTTP** アクションを使用する必要があります。 最適なユーザー エクスペリエンスを確保するためには、API の Swagger エンドポイントを公開する必要があります。 これにより、ロジック アプリ デザイナーで API の入出力を表示できるようになります。 Swagger がないと、デザイナーでは入出力を不透明な JSON オブジェクトとしてしか表示できません。

新しい `metadata.apiDefinitionUrl` プロパティを次の例に示します。

```
{
   "actions": {
        "mycustomAPI": {
            "type": "http",
            "metadata": {
              "apiDefinitionUrl": "https://mysite.azurewebsites.net/api/apidef/"  
            },
            "inputs": {
                "uri": "https://mysite.azurewebsites.net/api/getsomedata",
                "method": "GET"
            }
        }
    }
}
```

Web API を **App Service** でホストした場合、デザイナーで利用可能なアクションの一覧にその Web API が自動的に表示されます。 ホストしていない場合は、URL 内に直接貼り付ける必要があります。 Swagger エンドポイントは、ロジック アプリ デザイナー内で使用できるように、認証しない必要があります (ただし、Swagger でサポートされているいずれかの方法で API 自体をセキュリティで保護することはできます)。

### <a name="using-your-already-deployed-api-apps-with-2015-08-01-preview"></a>既にデプロイ済みの API アプリを 2015-08-01-preview で使用する
API アプリを事前にデプロイしてある場合、 **HTTP** アクションによってそのアプリを呼び出すことができます。

たとえば、Dropbox を使用してファイルを一覧表示する場合、**2014-12-01-preview** のスキーマ バージョン定義の内容は次のようになっていることがあります。

```
{
    "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2014-12-01-preview/workflowdefinition.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "/subscriptions/423db32d-...-b59f14c962f1/resourcegroups/avdemo/providers/Microsoft.AppService/apiapps/dropboxconnector/token": {
            "defaultValue": "eyJ0eX...wCn90",
            "type": "String",
            "metadata": {
                "token": {
                    "name": "/subscriptions/423db32d-...-b59f14c962f1/resourcegroups/avdemo/providers/Microsoft.AppService/apiapps/dropboxconnector/token"
                }
            }
        }
    },
    "actions": {
        "dropboxconnector": {
            "type": "ApiApp",
            "inputs": {
                "apiVersion": "2015-01-14",
                "host": {
                    "id": "/subscriptions/423db32d-...-b59f14c962f1/resourcegroups/avdemo/providers/Microsoft.AppService/apiapps/dropboxconnector",
                    "gateway": "https://avdemo.azurewebsites.net"
                },
                "operation": "ListFiles",
                "parameters": {
                    "FolderPath": "/myfolder"
                },
                "authentication": {
                    "type": "Raw",
                    "scheme": "Zumo",
                    "parameter": "@parameters('/subscriptions/423db32d-...-b59f14c962f1/resourcegroups/avdemo/providers/Microsoft.AppService/apiapps/dropboxconnector/token')"
                }
            }
        }
    }
}
```

同等の HTTP アクションを以下のように作成できます (ロジック アプリ定義の parameters セクションは元の状態のまま変更していません)。

```
{
    "actions": {
        "dropboxconnector": {
            "type": "Http",
            "metadata": {
              "apiDefinitionUrl": "https://avdemo.azurewebsites.net/api/service/apidef/dropboxconnector/?api-version=2015-01-14&format=swagger-2.0-standard"  
            },
            "inputs": {
                "uri": "https://avdemo.azurewebsites.net/api/service/invoke/dropboxconnector/ListFiles?api-version=2015-01-14",
                "method": "POST",
                "body": {
                    "FolderPath": "/myfolder"
                },
                "authentication": {
                    "type": "Raw",
                    "scheme": "Zumo",
                    "parameter": "@parameters('/subscriptions/423db32d-...-b59f14c962f1/resourcegroups/avdemo/providers/Microsoft.AppService/apiapps/dropboxconnector/token')"
                }
            }
        }
    }
}
```

これらのプロパティを&1; つずつ説明します。

| アクション プロパティ | 説明 |
| --- | --- |
| `type` |`APIapp` の代わりに `Http` を使用します |
| `metadata.apiDefinitionUrl` |ロジック アプリ デザイナーでこのアクションを使用する場合は、次の内容で構成されたメタデータ エンドポイントを含める必要があります。`{api app host.gateway}/api/service/apidef/{last segment of the api app host.id}/?api-version=2015-01-14&format=swagger-2.0-standard` |
| `inputs.uri` |次の内容で構成されています。`{api app host.gateway}/api/service/invoke/{last segment of the api app host.id}/{api app operation}?api-version=2015-01-14` |
| `inputs.method` |常に `POST` |
| `inputs.body` |API アプリのパラメーターと同じです |
| `inputs.authentication` |API アプリの認証と同じです |

この方法は、API アプリのアクションすべてに対して有効です。 ただし、このような以前の API アプリはもうサポートされていないため、前述した&2; つの方法のいずれか (マネージ API、またはカスタム Web API のホスト) に移行する必要があります。

## <a name="2-repeat-renamed-to-foreach"></a>2.Repeat から Foreach への名前変更

以前のスキーマ バージョンについては **Repeat** が理解しづらく、また **Repeat** は実際には for-each ループであることを正しく把握できないというフィードバックがお客様から数多く寄せられました。 そのため、名前を **Repeat** から **Foreach** に変更しました。 たとえば、これまでは次のように記述していました。

```
{
    "actions": {
        "pingBing": {
            "type": "Http",
            "repeat": "@range(0,2)",
            "inputs": {
                "method": "GET",
                "uri": "https://www.bing.com/search?q=@{repeatItem()}"
            }
        }
    }
}
```

これからは次のように記述します。

```
{
    "actions": {
        "pingBing": {
            "type": "Http",
            "foreach": "@range(0,2)",
            "inputs": {
                "method": "GET",
                "uri": "https://www.bing.com/search?q=@{item()}"
            }
        }
    }
}
```

これまで、`@repeatItem()` 関数は、反復処理中のアイテムを参照するために使用されていました。 これからは、この関数は簡略化されて `@item()` となります。 

### <a name="referencing-the-outputs-of-the-foreach"></a>Foreach の出力を参照する
さらに簡略化するために、**Foreach** アクションの出力は **repeatItems** というオブジェクトでラップされなくなります。 前述の repeat の出力は次のように記述していました。

```
{
    "repeatItems": [
        {
            "name": "pingBing",
            "inputs": {
                "uri": "https://www.bing.com/search?q=0",
                "method": "GET"
            },
            "outputs": {
                "headers": { },
                "body": "<!DOCTYPE html><html lang=\"en\" xml:lang=\"en\" xmlns=\"http://www.w3.org/1999/xhtml\" xmlns:Web=\"http://schemas.live.com/Web/\">...</html>"
            }
            "status": "Succeeded"
        }
    ]
}
```

これからは次のように記述します。

```
[
    {
        "name": "pingBing",
        "inputs": {
            "uri": "https://www.bing.com/search?q=0",
            "method": "GET"
        },
        "outputs": {
            "headers": { },
            "body": "<!DOCTYPE html><html lang=\"en\" xml:lang=\"en\" xmlns=\"http://www.w3.org/1999/xhtml\" xmlns:Web=\"http://schemas.live.com/Web/\">...</html>"
        }
        "status": "Succeeded"
    }
]
```

これらの出力を参照するとき、アクションの本体を参照するには、次のようにする必要がありました。

```
{
    "actions": {
        "secondAction": {
            "type": "Http",
            "repeat": "@outputs('pingBing').repeatItems",
            "inputs": {
                "method": "POST",
                "uri": "http://www.example.com",
                "body": "@repeatItem().outputs.body"
            }
        }
    }
}
```

今後は以下のように記述できます。

```
{
    "actions": {
        "secondAction": {
            "type": "Http",
            "foreach": "@outputs('pingBing')",
            "inputs": {
                "method": "POST",
                "uri": "http://www.example.com",
                "body": "@item().outputs.body"
            }
        }
    }
}
```

このような変更により、`@repeatItem()`、`@repeatBody()`、`@repeatOutputs()` 関数が削除されています。

## <a name="3-native-http-listener"></a>3.ネイティブ HTTP リスナー
現在、HTTP リスナーは組み込みの機能になっているため、HTTP リスナー API アプリをデプロイする必要はなくなりました。 ロジック アプリのエンドポイントを呼び出し可能にする方法の詳細については、 [こちら](../logic-apps/logic-apps-http-endpoint.md)を参照してください。 

この変更によって `@accessKeys()` 関数が削除され、代わりにエンドポイントの取得用として (必要な場合)、`@listCallbackURL()` 関数が用意されています。 また、現在はロジック アプリに少なくとも&1; つのトリガーを定義する必要があります。 ワークフローに対して `/run` を実行する場合は、`manual`、`apiConnectionWebhook`、`httpWebhook` のいずれかのトリガーが必要になります。

## <a name="4-calling-child-workflows"></a>4.子ワークフローの呼び出し
これまで、子ワークフローを呼び出すには、そのワークフローでアクセス トークンを取得し、呼び出し元とするロジック アプリの定義内にそのトークンを貼り付ける必要がありました。 新しいスキーマ バージョンを使用すると、Logic Apps エンジンにより、子ワークフローに対して実行時に SAS が自動生成されます。つまり、定義内にシークレットを貼り付ける必要はありません。  たとえば次のようになります。

```
"mynestedwf": {
    "type": "workflow",
    "inputs": {
        "host": {
            "id": "/subscriptions/xxxxyyyyzzz/resourceGroups/rg001/providers/Microsoft.Logic/mywf001",
            "triggerName": "myendpointtrigger"
        },
        "queries": {
            "extrafield": "specialValue"
        },
        "headers": {
            "x-ms-date": "@utcnow()",
            "Content-type": "application/json"
        },
        "body": {
            "contentFieldOne": "value100",
            "anotherField": 10.001
        }
    },
    "conditions": []
}
```

2 つ目の改善点は、受信要求へのフル アクセスが子ワークフローに付与されるようになったことです。 つまり、*queries* セクションと *headers* オブジェクトでパラメーターを渡すことができ、本体全体を自由に定義できるということになります。

最後に、子ワークフローに対して必要な変更があります。 これまでは子ワークフローを直接呼び出すことができましたが、これからは親から呼び出すためにワークフロー内にトリガー エンドポイントを定義する必要があります。 つまり、通常は **manual** 型のトリガーを追加して、それを親の定義内で使用することになります。 `host` プロパティには特別に `triggerName` が含まれていることに注意してください。これは、どのトリガーを呼び出すかを常に指定する必要があるためです。

## <a name="other-changes"></a>その他の変更点
### <a name="new-queries-property"></a>新しい queries プロパティ
現時点では、すべての種類のアクションで **queries**という新しい入力がサポートされています。 この入力は構造化オブジェクトにすることが可能で、文字列を手動で組み合わせる必要がありません。

### <a name="parse-function-renamed"></a>parse() 関数の名前変更
近日中にコンテンツの種類を追加するため、`parse()` 関数の名前を `json()` に変更しました。

## <a name="coming-soon-enterprise-integration-apis"></a>近日提供予定: Enterprise Integration API
Enterprise Integration API のマネージ バージョン (AS2 など) はまだ用意されていません。 [ロードマップ](http://www.zdnet.com/article/microsoft-outlines-its-cloud-and-server-integration-roadmap-for-2016/)に記載されているとおり、これらの API は近日提供予定です。 それまでは、既にデプロイ済みの API アプリの使用について上記で説明したように、HTTP アクションを通じて、デプロイ済みの既存の BizTalk API を使用してください。


