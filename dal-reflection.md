在写dal模块的时候，想要获取传入对象的属性，最后想到用反射，对应了cassandra表中的值——注解——getmethod的三者对应关系。

```java
    //cassandraClient.java
    public void setDeviceFactoryInfo(
            DeviceFactoryInfoEntity deviceFactoryInfo) throws Exception {
        if (deviceFactoryInfo == null) {
            return;
        }
        if (deviceFactoryInfo.getDeviceId() == null) {
            throw new NullPointerException("device id is null");
        }
        Update update = QueryBuilder.update("cloud", "device_factory_info");
        Map<String, Method> methodMap =
                getMethodMapping(deviceFactoryInfo.getClass());
        for (Map.Entry<String, Method> entry : methodMap.entrySet()) {
            Object value = entry.getValue().invoke(deviceFactoryInfo);
            if (value != null) {
                update.with(QueryBuilder.set(entry.getKey(), value));
            }
        }
        update.where(QueryBuilder.eq("device_id",
                deviceFactoryInfo.getDeviceId()));
        if (logger.isDebugEnabled()) {
            logger.debug("cassandra.update.cql:{}", update.getQueryString());
        }
        try {
            deviceDao.execute(update);
        } catch (Exception e) {
            logger.error("cassandra.update.failed:" + e.toString());
            throw e;
        }
    }

    public Map<String, Method> getMethodMapping(Class<?> clazz) {
        if (cachedName.containsKey(clazz)) {
            return cachedName.get(clazz);
        }

        Method[] methods = clazz.getMethods();
        Map<String, Method> methodMap = new HashMap<>();
        for (Method method : methods) {
            methodMap.put(method.getName(), method);
        }
        Map<String, Method> columnMap = new HashMap<>();
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            Column column = field.getAnnotation(Column.class);
            if (column != null &&
                    field.getAnnotation(PartitionKey.class) == null) {
                String fieldName = field.getName();
                String methodName = "get" + fieldName.substring(0, 1)
                        .toUpperCase() +
                        fieldName.substring(1, fieldName.length());
                columnMap.put(column.name(), methodMap.get(methodName));
            }
        }
        cachedName.put(clazz, columnMap);
        return columnMap;
    }
```
