---
---

# LdapConfig

```java
/**
 * <p>LDAP配置类</p>
 */
@Configuration
public class LdapConfig {

    @Value("${spring.ldap.urls}")
    private String ldapUrl;
    @Value("${spring.ldap.base}")
    private String base;
    @Value("${spring.ldap.username}")
    private String userName;
    @Value("${spring.ldap.password}")
    private String password;

    /**
     * <p>创建一个LDAP上下文源的Bean定义</p>
     * <p>LDAP上下文是专门用来连接LDAP库的环境</p>
     *
     * @return 返回{@link LdapContextSource}对象
     */
    @Bean
    public LdapContextSource ldapContextSource() {
        LdapContextSource source = new LdapContextSource();
        source.setUrl(this.ldapUrl);
        source.setUserDn(this.userName);
        source.setPassword(this.password);
        source.setBase(this.base);
//        source.setAuthenticationStrategy(new ExternalTlsDirContextAuthenticationStrategy());
        return source;
    }


    /**
     * <p>创建一个{@link LdapTemplate}的Bean定义</p>
     * <p>{@link LdapTemplate}是Spring提供的一个用于LDAP操作的工具类</p>
     *
     * @return 返回{@link LdapTemplate}对象
     */
    @Bean
    public LdapTemplate ldapTemplate() {
        return new LdapTemplate(ldapContextSource());
    }

    public AuthenticationManager getLdapAuthenticationManager() {
        LdapBindAuthenticationManagerFactory factory = new LdapBindAuthenticationManagerFactory(ldapContextSource());
        factory.setUserDnPatterns(
                "cn={0},ou=member,ou=user"
//                "uid={0},ou=member," + base
        );
        return factory.createAuthenticationManager();
    }

}
```
