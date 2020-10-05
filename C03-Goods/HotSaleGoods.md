# 热销商品

对于热销商品，前端在加载好页面后，向后端发送请求，获取属于当前商品类别（第三级类别）的热销商品。

#### 后端接口设计：

**请求方式**:  GET   `/categories/(?P<category_id>\d+)/hotskus/`

**请求参数**： 路径参数

| 参数        | 类型 | 是否必须 | 说明           |
| ----------- | ---- | -------- | -------------- |
| category_id | int  | 是       | 第三级商品类别 |

**返回数据**：JSON

```json
[
    {
        "id": 6,
        "name": "Apple iPhone 8 Plus (A1864) 256GB 深空灰色 移动联通电信4G手机",
        "price": "7988.00",
        "default_image_url": "http://image.meiduo.site:8888/group1/M00/00/02/CtM3BVrRbI2ARekNAAFZsBqChgk3141998",
        "comments": 1
    },
    {
        "id": 14,
        "name": "华为 HUAWEI P10 Plus 6GB+128GB 玫瑰金 移动联通电信4G手机 双卡双待",
        "price": "3788.00",
        "default_image_url": "http://image.meiduo.site:8888/group1/M00/00/02/CtM3BVrRdMSAaDUtAAVslh9vkK04466364",
        "comments": 1
    }
]
```

| 返回值            | 类型    | 是否必须 | 说明         |
| ----------------- | ------- | -------- | ------------ |
| id                | int     | 是       | 商品sku 编号 |
| name              | str     | 是       | 商品名称     |
| price             | decimal | 是       | 单价         |
| default_image_url | str     | 是       | 默认图片     |
| comments          | int     | 是       | 评论量       |

在goods/serializers.py中创建用于返回数据的序列化器

```python
class SKUSerializer(serializers.ModelSerializer):
    """
    SKU序列化器
    """
    class Meta:
        model = SKU
        fields = ('id', 'name', 'price', 'default_image_url', 'comments')
```

在goods/views.py中创建视图

```python
from rest_framework_extensions.cache.mixins import ListCacheResponseMixin

class HotSKUListView(ListCacheResponseMixin, ListAPIView):
    """
    热销商品, 使用缓存扩展
    """
    serializer_class = SKUSerializer
    pagination_class = None

    def get_queryset(self):
        category_id = self.kwargs['category_id']
        return SKU.objects.filter(category_id=category_id, is_launched=True).order_by('-sales')[:constants.HOT_SKUS_COUNT_LIMIT]
```

因为热销商品数据的接口经常被访问，所以添加使用drf-extensions提供的缓存支持。

当前返回的商品列表数据，所以使用ListCacheResponseMixin。

#### 前端

修改detail.js

```js
		// 获取热销商品数据
        get_hot_goods: function(){
            axios.get(this.host+'/categories/'+this.cat+'/hotskus/', {
                    responseType: 'json'
                })
                .then(response => {
                    this.hots = response.data;
                    for(var i=0; i<this.hots.length; i++){
                        this.hots[i].url = '/goods/' + this.hots[i].id + '.html';
                    }
                })
                .catch(error => {
                    console.log(error.response.data);
                })
        },
```

