


# 一、背景介绍

小到几个人的团队，大到上万人的公司，都开不开薪资的计算。
而薪资通常由考核制度组成，且变化频繁。
在这种情况下财务人员依据考核制度采用excel公式来完成薪资的计算是一件非常低效的事情。

 `excel-formula-engine` 的诞生就是为了解决这个问题,让薪资计算支持excel公式配置，薪资计算逻辑无需硬编码，
 实现薪资规则动态调整。



 `excel-formula-engine`是对阿里`QLExpress`脚本引擎语法函数的扩展:
 
 建议使用之前先对[QLExpress](https://github.com/alibaba/QLExpress) 做一个了解。
# 二、依赖和调用说明

```xml
                <dependency>
			<groupId>com.github.chengzh100</groupId>
			<artifactId>excel-module</artifactId>
			<version>1.0.2.RELEASE</version>
		</dependency>
```

```java
@Test
    public void testFunction() throws Exception {
        DefaultContext<String, Object> expressContext = new DefaultContext<String, Object>();
        ExpressRunner runner = new ExpressRunner(false, true);
        QLExpressExcelMoudle.registerModule(runner);
        String expresses = "IF(score >=100,'满分',IF(score>=80,'优秀','不够优秀'))";
        expressContext.put("score", 80);
        Object result = runner.execute(expresses, expressContext, null, false, true);

        System.out.println(result);
    }
```
# 三、综合示例

 ```java
 public class SalaryTest {
 
     @Test
     public void calculateTest() throws Exception {
 
         DefaultContext<String, Object> expressContext = new DefaultContext<String, Object>();
         ExpressRunner runner = new ExpressRunner(true, true);
         QLExpressExcelMoudle.registerModule(runner);
         expressContext.put("姓名", "程增辉");
         expressContext.put("姓名", "信息技术");
         expressContext.put("职位", "研发");
         expressContext.put("级别", "P1");
         expressContext.put("工号", "123456");
         expressContext.put("基本工资", new BigDecimal(10000));
         expressContext.put("岗位补贴", new BigDecimal(1000));
         expressContext.put("税前补差", new BigDecimal(100));
         expressContext.put("事假", 1);
         expressContext.put("病假", 2);
         expressContext.put("应出勤天数", 23);
         expressContext.put("月份", 7);
         Map<String, String> expressMap = new LinkedHashMap();
         expressMap.put("空调补助", "IF(OR(月份==6,月份==7),1200,0)");
         expressMap.put("绩效系数", "IF(AND(职位=='研发',级别=='P2'),1,0.8)");
         expressMap.put("绩效奖金", "IF(级别=='P1',5000*绩效系数,10000*绩效系数)");
         expressMap.put("事假扣除", "基本工资/应出勤天数*事假");
         expressMap.put("病假扣除", "基本工资/应出勤天数*病假*0.4");
         expressMap.put("应发工资", "ROUND(基本工资+岗位补贴+绩效奖金+税前补差-事假扣除+空调补助,2)");
         expressMap.entrySet().stream().forEach(en -> {
             try {
                 Object value = runner.execute(en.getValue(), expressContext, null, false, false);
                 System.out.println("薪资项:" + en.getKey() + ":==>" + value);
                 expressContext.put(en.getKey(), value);
             } catch (Exception e) {
                 e.printStackTrace();
             }
 
 
         });
 
 
     }
 
 }

 
 ```
 结果:
 ```text
薪资项:空调补助:==>1200
薪资项:绩效系数:==>0.8
薪资项:绩效奖金:==>4000.0
薪资项:事假扣除:==>434.7826086957
薪资项:病假扣除:==>347.82608695656
薪资项:应发工资:==>15865.22
 ```






