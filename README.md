# Gestion des Patients - Travail Pratique 3

## Description :
L'objectif de cette mission est de mettre en pratique mes connaissances dans le framework Spring Boot dans un cas réel, qui est une application de gestion des patients utilisant une base de données MySQL. Cette application contient deux utilisateurs avec deux rôles : UTILISATEUR et ADMIN.

Le rôle UTILISATEUR peut seulement consulter les données des patients.
Le rôle ADMIN peut gérer les données des patients.

## APIs :

| Method | Endpoint           | Description                      |
|--------|--------------------|----------------------------------|
| GET    | /                  | Redirect to /user/index          |
| GET    | /login             | Access to login page             |
| GET    | /user/index        | Access to home page              |
| GET    | /admin/formPatient | Access to add patient page       |
| GET    | /admin/edit        | Access to edit patient data page |
| POST   | /admin/delete      | Delete a patient record          |
| POST   | /admin/save        | Add a new patient                |
| POST   | /admin/edit        | Edit a patient record            |
| POST   | /admin/edit        | Edit a patient record            |

## Mise en œuvre du framework de sécurité :

Pour le framework de sécurité, j'ai choisi de mettre en œuvre l'authentification en mémoire pour fournir une prise en charge de l'authentification par nom d'utilisateur/mot de passe qui est stockée en mémoire en utilisant Bcrypt pour le chiffrement.

```java
public InMemoryUserDetailsManager inMemoryUserDetailsManager(PasswordEncoder passwordEncoder){
        String encodedPassword = passwordEncoder.encode("----");
        return new InMemoryUserDetailsManager(
                User.withUsername("redar").password(encodedPassword).roles("USER").build(),
                User.withUsername("admin").password(encodedPassword).roles("USER","ADMIN").build()
        );
    }
```

J'ai également créé une configuration de chaîne de filtres de sécurité. Elle définit diverses mesures de sécurité telles que la connexion par formulaire avec une page de connexion désignée, la fonctionnalité de se souvenir de moi, et la gestion des scénarios d'accès refusé. De plus, elle spécifie des règles d'autorisation basées sur les rôles des utilisateurs, en restreignant l'accès à certains endpoints aux utilisateurs ayant des rôles spécifiques, tout en exigeant une authentification pour les autres endpoints.

````java
public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
        return httpSecurity
                .formLogin(form -> form.loginPage("/login").permitAll())
                .rememberMe(r -> r.alwaysRemember(true))
                .exceptionHandling(ex -> ex.accessDeniedPage("/404"))
                .authorizeHttpRequests(ar->ar.requestMatchers("/deletePatient/**").hasRole("ADMIN"))
                .authorizeHttpRequests(ar->ar.requestMatchers("/admin/**").hasRole("ADMIN"))
                .authorizeHttpRequests(ar->ar.requestMatchers("/user/**").hasRole("USER"))
                .authorizeHttpRequests(ar -> ar.requestMatchers("/webjars/**"))
                .authorizeHttpRequests(ar->ar.anyRequest().authenticated())
                .build();
    }
````

##  validation de donnees :
Validation des données
Il est crucial de vérifier si les entrées de l'utilisateur sont valides avant de les stocker dans notre base de données, et pour cela, j'utilise la dépendance spring-boot-starter-validation.
```java
public class Patient {
    @Id @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @NotEmpty
    private String fn;
    @NotEmpty
    private String ln;
    @Temporal(TemporalType.DATE)
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date birthday;
    private boolean sick;
    @DecimalMin("100")
    private int score;
}
```

## Demo :
<p align="center">
    <img src="images/img.png" alt="Description of the image" />
    <br />
    <em>Login page</em>
</p>
<p align="center">
    <img src="images/img_1.png" alt="Description of the image"  />
    <br />
    <em>Home page</em>
</p>
<p align="center">
    <img src="images/img_3.png" alt="Description of the image" />
    <br />
    <em>Adding a new patient's data</em>
</p>
<p align="center">
    <img src="images/img_4.png" alt="Description of the image"  />
    <br />
    <em>Editing a record</em>
</p>
<p align="center">
    <img src="images/img_2.png" alt="Description of the image" />
    <br />
    <em>Deleting a record</em>
</p>
<p align="center">
    <img src="images/img_5.png" alt="Description of the image" />
    <br />
    <em>Validation errors</em>
</p>
<p align="center">
    <img src="images/img_6.png" alt="Description of the image" />
    <br />
    <em>Access denied</em>
</p>
