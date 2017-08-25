---
layout: post
title:  "使用 selenium2 进行测试"
date:   2017-08-21
categories: selenium selenium2 前端
---

# 使用 selenium2 进行测试

1. 下载浏览器对应版本的 WebDriver.
Chrome 的下载地址是 http://chromedriver.storage.googleapis.com/index.html

2. 创建 maven quick start 项目

修改 pom.xml

```
<!-- https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java -->
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>3.4.0</version>
</dependency>

```

3. 写第一段测试代码

```
public class App {
    static WebDriver driver;
    public static void main(String[] args) {
        System.setProperty("webdriver.chrome.driver","F:\\Tools\\webdriver\\chromeDriver2.30\\chromedriver.exe");
        driver = new ChromeDriver();
        openWebsite();

    }

    private static void openWebsite() {
        Navigation navigation = driver.navigate();
        navigation.to("http://www.bing.com");
    }
}

```

4. 页面元素操作

```
private static void searchOnBing() {
    Navigation navigation = driver.navigate();
    navigation.to("http://www.bing.com");

    Thread.sleep(3000);
    WebElement bingSearchBox = driver.findElement(By.id("sb_form_q"));
    bingSearchBox.sendKeys("gitlab merge request 删除分支");
    bingSearchBox.submit();

}

```

此时有可能代码报错，提示 `Only local connections are allowed.`，此提示可以直接忽略。

