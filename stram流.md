## Arrays.toList()	将数组转换为list集合

## 将JSON格式的字符串转换为map集合

```
Map<String, String> map = JSON.parseObject(sku.getSpec(), Map.class);
```

## StringBuffer 字符串拼接
示例

```
StringBuilder url = new StringBuilder("/search.do?");
        for (String key : searchMap.keySet()) {
            url.append("&"+ key + "=" + searchMap.get(key));
        }
```

## thymeleaf:

表达式：
内置对象maps.containsKey 判断map中是否有该key
示例
th:if="!${#maps.containsKey(searchMap,'category')}"

字符串大小
categoryList.size()

Long包装类型取int值
示例
Long totalPage = (Long) result.get("totalPage");//得到总页码
int startPage = 1;//开始页码
int endPage = totalPage.intValue();//结束页码

## 将验证码（字符串）存储到redis
​        redisTemplate.boundValueOps("code_"+phoneNum).set(checkCode+"");
​        redisTemplate.boundValueOps("code_"+phoneNum).expire(5, TimeUnit.MINUTES);//5分钟失效

 ## BCrypt密码加密
​        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
​        String BCryptPassword = encoder.encode(user.getPassword());
​        user.setPassword(BCryptPassword);
​        System.out.println("BCryptPassword: "+BCryptPassword);



## Vue中处理小数的函数toFixed

```html
{{(cart.Item.price/100).toFixed(2)}}
```

## stream过滤器

```java
public List<Map<String, Object>> findCartFromRedis(String username) {
        List<Map<String, Object>> cartList = (List<Map<String, Object>>) redisTemplate.boundHashOps(CacheKey.CART_LIST).get(username);
        if (cartList == null) {//该用户名购物车为空
            System.out.println("carList is empty 该用户名购物车为空");
            return new ArrayList<>();//初始化赋值
        }
        return cartList;
    }
//删除购物车中选中的商品 采用stream流放行未被选中的
List<Map<String, Object>> cartList = 
    findCartFromRedis(username).stream().filter(
    	cart -> (Boolean) cart.get("checked") == false).collect(Collectors.toList());
```



```java
public int preferential(String username) {
    //获取选中的商品列表
    List<OrderItem> itemList =
            findCartFromRedis(username).stream().filter(
                    cart -> (Boolean) cart.get("checked") == true).map(
                    cart -> (OrderItem) cart.get("Item")).collect(Collectors.toList());
    //对商品列表进行分类聚合
    Map<Integer, IntSummaryStatistics> categoryMoneyMap =
            itemList.stream().collect(
                    Collectors.groupingBy(OrderItem::getCategoryId3, Collectors.summarizingInt(OrderItem::getMoney)));
    //循环遍历结果进行累加
    int allPerMoney = 0; //累计优惠金额
    for (Integer categoryId : categoryMoneyMap.keySet()) {
        int money = (int) categoryMoneyMap.get(categoryId).getSum();
        int preMoney = preferentialService.findPreMoney(categoryId, money);//获取优惠金额
        System.out.println("categoryId(分类): " + categoryId + " money(消费金额): " + money + " perMoney(优惠金额): "+preMoney);
        allPerMoney += preMoney;
    }
    return allPerMoney;
}
```



在使用通用Mapper进行自定义sql语句时要使用mysql的下划线字段，不能按驼峰格式写

```java
/**
 * 更新销量
 * @param skuId
 * @param num
 */
@Update("update tb_sku set sale_num = sale_num + #{saleNum} where id = #{skuId}")
public void updateSaleNum(@Param("skuId") String skuId, @Param("saleNum") Integer num);
```

