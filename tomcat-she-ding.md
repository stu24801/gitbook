# Tomcat 設定

## &#x20;**Failed to destroy end point associated with ProtocolHandler\["ajp-nio-8009"]**

**解法:**

tomcat中 Server Locations

默認是選擇  Use workspace metadata(dose not modify Tomcat installation)

然後修改爲 Use Tomcat installation(takes control of Tomcat installation)

可能你打開，發現是灰色的不能修改，那是因爲Server中有部署項目，把項目都移除，然後再啓動就可以操作了。

