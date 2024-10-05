# 1.  多表连接字段冲突，导致无法返回正确内容

## 1. 直接使用列别名

```xml
    <select id="selectMenuById" resultMap="menuBoList">
        SELECT m.id, m.name, m.url, mm.id, mm.title, mm.icon
        FROM bbs.menu m
                 LEFT JOIN bbs.menu_meta mm ON m.id = mm.menu_id
        WHERE m.id = #{id};
    </select>
```

# 2. 使用Sql标签别名

