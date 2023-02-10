# 第一题解决方案：
设计区域表、省、市表、区域类别特征表根据类别特征及等级不同(值越大优先级越高)，按用户所在城市，查找其所在区域，按其区域特征查询关联的商品类别，优先查询级别高的商品。
简单的设计表
-- 区域表 --
CREATE TABLE `cy_area` (
`a_id` varchar(50) NOT NULL COMMENT '区域编号',
`a_name` varchar(50) NOT NULL COMMENT '区域名称', 
`a_desc` varchar(200) NOT NULL COMMENT '区域描述',
PRIMARY KEY (`a_id`) ) 
ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci

-- 省份表 -- 
CREATE TABLE `cy_province` (
`p_id` varchar(50) NOT NULL COMMENT '省份编号',
`p_name` varchar(50) NOT NULL COMMENT '省份名称',
`a_id` varchar(50) COMMENT '区域编号',
PRIMARY KEY (`p_id`), 
CONSTRAINT `province_a_id_foreign` FOREIGN KEY (`a_id`) REFERENCES `cy_area` (`a_id`) ON DELETE CASCADE ON UPDATE CASCADE 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci 

-- 城市表 --
CREATE TABLE `cy_city` ( `c_id` varchar(50) NOT NULL COMMENT '城市编号', `c_name` varchar(50) NOT NULL COMMENT '城市名称',
`p_id` varchar(50) COMMENT '省份编号', 
PRIMARY KEY (`c_id`), 
CONSTRAINT `city_p_id_foreign` FOREIGN KEY (`p_id`) REFERENCES `cy_province` (`p_id`) ON DELETE CASCADE ON UPDATE CASCADE 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci

-- 区域类别特征表 -- 
CREATE TABLE `cy_area_categories` ( 
`id` int(10) unsigned NOT NULL AUTO_INCREMENT, 
`a_id` varchar(50) COMMENT '区域编号',
`categories_id` int(10) COMMENT '商品分类编号', 
`level` int(10) COMMENT '特征级别', 
`ac_desc` varchar(50) NOT NULL COMMENT '特征描述',
PRIMARY KEY (`id`), 
CONSTRAINT `cates_aid_foreign` FOREIGN KEY (`a_id`) REFERENCES `cy_area` (`a_id`) ON DELETE CASCADE ON UPDATE CASCADE, CONSTRAINT `categories_id_foreign` FOREIGN KEY (`categories_id`) REFERENCES `cy_categories` (`id`) ON DELETE CASCADE ON UPDATE CASCADE 
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci 

mapper.xml如下查询SQL
商品表和库存表外连接区域特征表查询商品启用的信息，按区域特征等级降序分页查询
<select id="selectByPage" resultType="goods">
select g.* from cy_goods g inner join cy_goods_sku k on g.id=k.goods_id left join cy_area_categories c on c.categories_id=g.cate_id and c.a_id=(select a_id from cy_province where p_id=#{pid}) order by c.level desc
limit ${limit},${size} 
</select>

前端通过当前用户的地理位置，获取所在省份信息，讲省份编号传递回后台，实现商品最优查询
var pid = //获取当前用户所在(城市)省份编号 
axios("goods/selectByPage", `pid=${pid}`).then(res=>{ this.list = res.data; 
})

第二题解决方案：
尽量不更改原有的service和controller实现，创建新的GoodsServiceImpl2，继承原有的service，重新商品分页查询方法，商品controller用@Resource注解，讲最新的service注入进来，实现更改前后的效果对比
