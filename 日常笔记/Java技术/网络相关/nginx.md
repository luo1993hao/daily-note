### location
**匹配原则**

- 总的来说，匹配顺序
长，普通location > 短，普通location >正则匹配  ，一般来说，会先普通location然后正则location，除非碰到严格匹配与"^~"
-  正则与正则的匹配规则：物理顺序
- 正则 location 匹配让步普通 location 的严格精确匹配结果；但覆盖普通 location 的最大前缀匹配结果
- 普通的location匹配与编辑顺序无关
- "="为严格匹配，例如“location = / {} ”，只能匹配 http://host:port/ 请求，并且禁止搜索正则
