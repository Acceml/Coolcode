hibernate中更新非空域；传入一个对象，这个对象中有的域可能是null，但是我并不想覆盖原来的数据库中的有值的域。
网络上的方法（设置`dynamic-update`）不适用这种场景：
[设置dynamic-update](http://www.mkyong.com/hibernate/hibernate-dynamic-update-attribute-example/)
自己用一种比较蠢的方法：
```java
    public void setAccount(Account a) throws HibernateException {
        try {
            Account tmp = (Account) session.
                    get(Account.class, a.getAccountId());
            tmp.setEmail(getNotNull(a.getEmail(), tmp.getEmail()));
            tmp.setMobile(getNotNull(a.getMobile(), tmp.getMobile()));
            tmp.setPassword(getNotNull(a.getPassword(), tmp.getPassword()));
            tmp.setStatus(getNotNull(a.getStatus(), tmp.getStatus()));
            tmp.setRegisterDate(getNotNull(a.getRegisterDate(),
                    tmp.getRegisterDate()));
            tmp.setLockEndDate(getNotNull(a.getLockEndDate(),
                    tmp.getLockEndDate()));
            tmp.setVersion(getNotNull(a.getVersion(), tmp.getVersion()));
            session.beginTransaction();
            session.update(tmp);
            session.getTransaction().commit();
        } catch (HibernateException e) {
            logger.error(e.toString());
            throw e;
        }
    }

    public static <T> T getNotNull(T a, T b) {
        return b != null && a != null && !a.equals(b) ? a : b;
    }
```
另外，`equals`在用的时候必须先判断一下非空。见[`api`](https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#equals(java.lang.Object))
