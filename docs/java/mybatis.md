
## 分页功能
- 设置拦截器：

```java
@Configuration
public class MpCongfig {
	@Bean
	public MybatisPlusInterceptor pageInterceptor(){
		MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 加入分页拦截器
		interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
		return interceptor;
	}
}
```

- 业务层实现分页起

```java
public R<Page> page(int page,int pageSize){
    //分页构造器
    Page<Category> pageInfo = new Page<>(page,pageSize);
    //条件构造器
    LambdaQueryWrapper<Category> queryWrapper = new LambdaQueryWrapper<>();
    //添加排序条件，根据sort进行排序
    queryWrapper.orderByAsc(Category::getSort);

    //分页查询
    categoryService.page(pageInfo,queryWrapper);
    return R.success(pageInfo);
}
```

## 常用注解说明
- `@TableId`: 指定主键，其中`type`属性可以指定生成策略
- `@TableField`: 建立数据库字段名和实体类名的映射
  - `exist`属性可以指定是否为数据库字段
  - `select`属性可以指定是否为查询字段
  - `update`属性可以指定是否为更新字段
  - `insert`属性可以指定是否为插入字段
  - `fill`属性可以指定是否为自动填充字段
- `@Version`: 乐观锁注解，用于指定乐观锁的字段
- `@TableLogic`: 逻辑删除注解，用于指定逻辑删除的字段
  - `value`属性可以指定逻辑删除的值
  - `delval`属性可以指定逻辑未删除的值