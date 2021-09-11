---
layout: post
title: "Elasticsearch"
subtitle: "Springboot中使用Elasticsearch"
date: 2021-9-11 16:24:29
background: '/img/posts/06.jpg'
---
# Elasticsearch介绍：  
Elasticsearch 是一个分布式、可扩展、实时的搜索与数据分析引擎。 它能从项目一开始就赋予你的数据以搜索、分析和探索的能力，可用于实现全文搜索和实时数据统计。  
主要用于模糊搜索，相较于mysql中的%...%搜索方式（当使用右匹配时不会通过索引进行匹配，效率会大大降低），可以提高在数据量较大时的搜索效率。
当写入数据到Elasticsearch中时，会使用指定算法进行分词，对分词后得到的结果建立索引。  

<img src="..\img\posts\elasticsearch.PNG" style="zoom:50%;" />

# 安装和使用：
## Pom中添加dependence：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```
## 确认版本：
打开Springboot项目Project目录中External Libraries，找到elasticsearch相关包，确认相关版本号。
## 下载对应版本的elasticsearch和Kibana：
### elasticsearch：
访问该地址（记得修改为对应版本号，此处使用的是elasticsearch7.9.3）：
[https://www.elastic.co/cn/downloads/past-releases/elasticsearch-7-9-3](https://www.elastic.co/cn/downloads/past-releases/elasticsearch-7-9-3)  
之后解压到自定义的位置。
### 由于elasticsearch默认分词是针对英文，所以需要添加中文分词插件：
在终端打开elasticsearch文件夹下的bin目录，执行

```
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.9.3/elasticsearch-analysis-ik-7.9.3.zip
```
注意修改命令中对应的版本号，要与elasticsearch版本对应。  
之后在使用elasticsearch时，运行bin目录中的elasticsearch.bat文件，即可开启elasticsearch（不开启会出现connection timeout exception）。
### 下载kibana：  
下载Kibana,作为访问Elasticsearch的客户端，请下载对应版本的zip包，并解压到指定目录，下载地址：
[https://artifacts.elastic.co/downloads/kibana/kibana-7.9.3-windows-x86_64.zip](https://artifacts.elastic.co/downloads/kibana/kibana-7.9.3-windows-x86_64.zip)  
之后解压到自定义位置，运行bin目录中的kibana.bat,之后访问localhost:5601即可打开kibana用户界面。  
# 与Springboot整合：
## 常用注解：
* @Document:  
indexName:相当于表名
* @Id：与mysql中id作用类似
* @Field：相当于mysql中的column，需要指定fieldType，使用模糊搜索时还需要添加analyzer。不需要中文分词的字段设置成@Field(type = FieldType.Keyword)类型，需要中文分词的设置成@Field(analyzer = "ikmaxword",type = FieldType.Text)类型。
## application.yml中spring下添加配置：

```
  data:
    elasticsearch:
      repositories:
        enabled: true
  elasticsearch:
    rest:
      uris: http://localhost:9200
```
## 添加商品文档对象EsProduct：

```
package com.jadon.mall.nosql.elasticsearch.document;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

import java.io.Serializable;
import java.math.BigDecimal;
import java.util.List;

@Document(indexName = "pms_product", replicas = 0)
public class EsProduct implements Serializable {
    private static final Logger LOGGER = LoggerFactory.getLogger(EsProduct.class);
    private static final long serialVersionUID = -1L;
    @Id
    private Long id;
    @Field(type = FieldType.Keyword)
    private String productSn;
    private Long brandId;
    @Field(type = FieldType.Keyword)
    private String brandName;
    private Long productCategoryId;
    @Field(type = FieldType.Keyword)
    private String productCategoryName;
    private String pic;
    @Field(analyzer = "ik_max_word", type = FieldType.Text)
    private String name;
    @Field(analyzer = "ik_max_word", type = FieldType.Text)
    private String subTitle;
    @Field(analyzer = "ik_max_word", type = FieldType.Text)
    private String keywords;
    private BigDecimal price;
    private Integer sale;
    private Integer newStatus;
    private Integer recommandStatus;
    private Integer stock;
    private Integer promotionType;
    private Integer sort;
    @Field(type = FieldType.Nested)
    private List<EsProductAttributeValue> attrValueList;
    
    // setters and getters...
}

```
## 添加商品属性参数对象EsProductAttributeValue：

```
package com.jadon.mall.nosql.elasticsearch.document;

import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

import java.io.Serializable;

public class EsProductAttributeValue implements Serializable {
    private static final long serialVersionUID = 1L;
    private Long id;
    private Long productAttributeId;
    //属性值
    @Field(type = FieldType.Keyword)
    private String value;
    //属性参数：0->规格；1->参数
    private Integer type;
    @Field(type = FieldType.Keyword)
    private String name;
    
    // setters and getters...
}
```

## 添加EsProductRepository接口用于操作Elasticsearch：

```
package com.jadon.mall.nosql.elasticsearch.repository;

import com.jadon.mall.nosql.elasticsearch.document.EsProduct;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

public interface EsProductRepository extends ElasticsearchRepository<EsProduct, Long> {

    /**
     * 搜索查询
     *
     * @param name     商品名称
     * @param subTitle 商品标题
     * @param keywords 商品关键字
     * @param page     分页信息
     * @return
     */
    Page<EsProduct> findByNameOrSubTitleOrKeywords(String name, String subTitle, String keywords, Pageable page);
}
```
## 创建EsProductDao用于从mysql数据库中获取全部商品：

EsProductDao
```
package com.jadon.mall.dao;

import com.jadon.mall.nosql.elasticsearch.document.EsProduct;
import org.apache.ibatis.annotations.Param;

import java.util.List;

public interface EsProductDao {
    List<EsProduct> getAllEsProductList(@Param("id") Long id);
}
```
EsProductDao.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jadon.mall.dao.EsProductDao">
    <resultMap id="esProductListMap" type="com.jadon.mall.nosql.elasticsearch.document.EsProduct" autoMapping="true">
        <id column="id" jdbcType="BIGINT" property="id"/>
        <collection property="attrValueList" columnPrefix="attr_"
                    ofType="com.jadon.mall.nosql.elasticsearch.document.EsProductAttributeValue">
            <id column="id" property="id" jdbcType="BIGINT"/>
            <id column="id" property="id" jdbcType="BIGINT"/>
            <result column="product_attribute_id" property="productAttributeId" jdbcType="BIGINT"/>
            <result column="value" property="value" jdbcType="VARCHAR"/>
            <result column="type" property="type"/>
            <result column="name" property="name"/>
        </collection>
    </resultMap>
    <select id="getAllEsProductList" resultMap="esProductListMap">
        select
        p.id id,
        p.product_sn productSn,
        p.brand_id brandId,
        p.brand_name brandName,
        p.product_category_id productCategoryId,
        p.product_category_name productCategoryName,
        p.pic pic,
        p.name name,
        p.sub_title subTitle,
        p.price price,
        p.sale sale,
        p.new_status newStatus,
        p.recommand_status recommandStatus,
        p.stock stock,
        p.promotion_type promotionType,
        p.keywords keywords,
        p.sort sort,
        pav.id attr_id,
        pav.value attr_value,
        pav.product_attribute_id attr_product_attribute_id,
        pa.type attr_type,
        pa.name attr_name
        from pms_product p
        left join pms_product_attribute_value pav on p.id = pav.product_id
        left join pms_product_attribute pa on pav.product_attribute_id= pa.id
        where delete_status = 0 and publish_status = 1
        <if test="id!=null">
            and p.id=#{id}
        </if>
    </select>
</mapper>
```
## 创建EsProductService：

```
package com.jadon.mall.service;

import com.jadon.mall.nosql.elasticsearch.document.EsProduct;
import org.springframework.data.domain.Page;

import java.util.List;

public interface EsProductService {
    /**
     * 从数据库导入所有商品到ES
     */
    int importAll();

    /**
     * 根据id删除商品
     */
    void delete(Long id);

    /**
     * 根据id创建商品
     */
    EsProduct create(Long id);

    /**
     * 批量删除商品
     */
    void delete(List<Long> ids);

    /**
     * 根据关键字搜索名称或者副标题
     */
    Page<EsProduct> search(String keyword, Integer pageNum, Integer pageSize);
}
```
## 实现EsProductService：

```
package com.jadon.mall.service.impl;

import com.jadon.mall.dao.EsProductDao;
import com.jadon.mall.nosql.elasticsearch.document.EsProduct;
import com.jadon.mall.nosql.elasticsearch.repository.EsProductRepository;
import com.jadon.mall.service.EsProductService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

@Service
public class EsProductServiceImpl implements EsProductService {
    private static final Logger LOGGER = LoggerFactory.getLogger(EsProductServiceImpl.class);
    @Autowired
    private EsProductDao productDao;
    @Autowired
    private EsProductRepository productRepository;

    /**
     * 从数据库导入所有商品到ES
     */
    @Override
    public int importAll() {
        List<EsProduct> esProductList = productDao.getAllEsProductList(null);
        Iterable<EsProduct> esProductIterable = productRepository.saveAll(esProductList);
        Iterator<EsProduct> iterator = esProductIterable.iterator();
        int result = 0;
        while (iterator.hasNext()) {
            result++;
            iterator.next();
        }
        return result;
    }

    /**
     * 根据id删除商品
     */
    @Override
    public void delete(Long id) {
        productRepository.deleteById(id);
    }

    /**
     * 根据id创建商品
     */
    @Override
    public EsProduct create(Long id) {
        EsProduct result = null;
        List<EsProduct> esProductList = productDao.getAllEsProductList(id);
        if (esProductList.size() > 0) {
            EsProduct esProduct = esProductList.get(0);
            result = productRepository.save(esProduct);
        }
        return result;
    }

    /**
     * 批量删除商品
     */
    @Override
    public void delete(List<Long> ids) {
        if (!CollectionUtils.isEmpty(ids)) {
            List<EsProduct> esProductList = new ArrayList<>();
            for (Long id : ids) {
                EsProduct esProduct = new EsProduct();
                esProduct.setId(id);
                esProductList.add(esProduct);
            }
            productRepository.deleteAll(esProductList);
        }
    }

    /**
     * 根据关键字搜索名称或者副标题
     */
    @Override
    public Page<EsProduct> search(String keyword, Integer pageNum, Integer pageSize) {
        Pageable pageable = PageRequest.of(pageNum, pageSize);
        return productRepository.findByNameOrSubTitleOrKeywords(keyword, keyword, keyword, pageable);
    }
}

```

## 实现Controller：

```
package com.jadon.mall.controller;

import com.jadon.mall.common.api.CommonPage;
import com.jadon.mall.common.api.CommonResult;
import com.jadon.mall.nosql.elasticsearch.document.EsProduct;
import com.jadon.mall.service.EsProductService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@Controller
@Api(tags = "EsProductController", description = "搜索商品管理")
@RequestMapping("/esProduct")
public class EsProductController {
    @Autowired
    private EsProductService esProductService;

    @ApiOperation(value = "导入所有数据库中商品到ES")
    @RequestMapping(value = "/importAll", method = RequestMethod.POST)
    @ResponseBody
    public CommonResult<Integer> importAllList() {
        int count = esProductService.importAll();
        return CommonResult.success(count);
    }

    @ApiOperation(value = "根据id删除商品")
    @RequestMapping(value = "/delete/{id}", method = RequestMethod.GET)
    @ResponseBody
    public CommonResult<Object> delete(@PathVariable Long id) {
        esProductService.delete(id);
        return CommonResult.success(null);
    }

    @ApiOperation(value = "根据id批量删除商品")
    @RequestMapping(value = "/delete/batch", method = RequestMethod.POST)
    @ResponseBody
    public CommonResult<Object> delete(@RequestParam("ids") List<Long> ids) {
        esProductService.delete(ids);
        return CommonResult.success(null);
    }

    @ApiOperation(value = "根据id创建商品")
    @RequestMapping(value = "/create/{id}", method = RequestMethod.POST)
    @ResponseBody
    public CommonResult<EsProduct> create(@PathVariable Long id) {
        EsProduct esProduct = esProductService.create(id);
        if (esProduct != null) {
            return CommonResult.success(esProduct);
        } else {
            return CommonResult.failed();
        }
    }

    @ApiOperation(value = "简单搜索")
    @RequestMapping(value = "/search/simple", method = RequestMethod.GET)
    @ResponseBody
    public CommonResult<CommonPage<EsProduct>> search(@RequestParam(required = false) String keyword,
                                                      @RequestParam(required = false, defaultValue = "0") Integer pageNum,
                                                      @RequestParam(required = false, defaultValue = "5") Integer pageSize) {
        Page<EsProduct> esProductPage = esProductService.search(keyword, pageNum, pageSize);
        return CommonResult.success(CommonPage.restPage(esProductPage));
    }

}

```
## 之后运行elasticsearch.bat,并启动application,即可使用swagger-ui进行接口测试。