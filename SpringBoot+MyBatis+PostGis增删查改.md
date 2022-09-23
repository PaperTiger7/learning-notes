# SpringBoot+MyBatis+PostGis增删查改

#### 1、几何图形数据

- 一个**点**（POINT）
- 一条**线**（LINESTRING）
- 一个**多边形**（POLYGON）
- 一个**内含空洞的多边形**（POLYGON with a hole）
- 一个**图形集合**（COLLECTION）

#### 2、元数据表

PostGIS提供了两张表用于追踪和报告数据库中的几何图形

- spatial_ref_sys：定义了数据库已知的所有**空间参照系统**
- geometry_columns：数据库中所有空间数据表的描述信息

![image-20220905112832768](C:\Users\wb\AppData\Roaming\Typora\typora-user-images\image-20220905112832768.png)

#### 3、表示真实世界对象

- **ST_GeometryType(geometry)**  ——  返回几何图形的类型
- **ST_NDims(geometry)**  ——  返回几何图形的维数
- **ST_SRID(geometry)**  ——  返回几何图形的空间参考标识码

#### 4、几何图形输入输出

- **ST_GeomFromText(text, srid)**  ——  返回geometry
- **ST_AsText(geometry)**  ——  返回text

- **ST_AsGeoJSON(geometry)**  ——  返回text
- **ST_GeomFromGeoJSON(geomJSON)**  ——  返回geometry
- **ST_Perimeter(geometry)** —— 返回几何面周长
- **ST_Area(geometry)** —— 返回几何面面积

#### 5、空间关系

- **ST_Intersects**、**ST_Crosses**、**ST_Overlaps**测试几何图形是否相交

​	如果两个图形有相同的部分，即如果它们的边界或内部相交，则**ST_Intersects(geometry A, geometry B)**返回TRUE。**intersect**测试可以使用**空间索引**。

```sql
SELECT name, boroname
FROM nyc_neighborhoods
WHERE ST_Intersects(geom, ST_GeomFromText('POINT(583571 4506714)',4326));
```

#### 6、增删改查

##### 6.1 数据增加

controller层：

```java
	@PostMapping
    public MapElement addMapElement(@RequestBody MapElement mapElement){
        mapElement.setGeoStr(geometryToString(mapElement.getLongitude(), mapElement.getLatitude()));
        mapService.addMapElement(mapElement);
        Long id = mapElement.getId();
        return mapService.findById(id);
    }

    private String geometryToString(double longitude, double latitude){
            String geoStr = "POINT" + "(" + longitude + " " + latitude + ")";
            return geoStr;
    }
```

mybatis层：

```sql
<insert id="addMapElement" 		parameterType="com.honorzhang.postgresql.model.MapElement" useGeneratedKeys="true" keyProperty="id">
	<!--PostgreSQL所特有的生成自增主键的方法-->
	<selectKey keyProperty="id" resultType="Long" order="BEFORE">
		SELECT nextval('map_elements_id_seq'::regclass)
	</selectKey>
        insert into map_elements(name, longitude, latitude, element_location)
        values (#{name}, #{longitude}, #{latitude}, ST_GeomFromGeoJSON(#{geoStr}))
</insert>
```

##### 6.2 数据删除

controller层：

```java
@DeleteMapping("/{id}")
    public Boolean deleteMapElement(@PathVariable Long id){
        Boolean deleteMapElementSuccess = true;
        try{
            mapService.deleteMapElement(id);
        }catch (Exception e){
            log.info("删除失败：" + e);
            deleteMapElementSuccess = false;
        }
        return deleteMapElementSuccess;
    }
```

mybatis层：

```sql
<delete id="deleteMapElement" parameterType="Long">
	delete from map_elements where id = #{id}
</delete>
```

##### 6.3 数据修改

controller层：

```java
	@PutMapping()
    public MapElement updateMapElement(@RequestBody MapElement mapElement){
        mapElement.setGeoStr(geometryToString(mapElement.getLongitude(), mapElement.getLatitude()));
        mapService.updateMapElement(mapElement);
        Long id = mapElement.getId();
        return mapService.findById(id);
    }
```

mybatis层：

```
<update id="updateMapElement"  parameterType="com.honorzhang.postgresql.model.MapElement" useGeneratedKeys="true" keyProperty="id" >
	UPDATE map_elements
	<trim prefix="set" suffixOverrides=",">
        <if test="name!=null">name = #{name},</if>
        <if test="longitude!=null">longitude = #{longitude},</if>
        <if test="latitude!=null">latitude = #{latitude},</if>
        <if test="geoStr!=null">element_location = ST_GeomFromText(#{geoStr}, 4326),</if>
	</trim>
	WHERE id=#{id}
</update>
```

