Spring Security – Storing User Credential in MySQL Database

1. Configure mySQL database
To configure the database, you need to add the necessary mySQL connection properties in the application.properties file.

All the properties, you need to add are given as:

spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.password=root
spring.datasource.username=root
spring.datasource.url=jdbc:mysql://localhost:3307/usersdb
spring.jpa.hibernate.ddl-auto=update

You may have to include ?serverTimezone=UTC to the url

Notice that the name of the database is usersdb and the port number is 3307.

Now, this databases should exist. (you should create it in mySQL and so you must have mySQL running in your system)

Also, you need to create a table with three fields:

id
username
password
Add a test record into the database. For me I added two records:

id: 1;   username: Hercules;   password: gift
id: 2;   username: Archilles;   password: gold
 

2. Ensure the necessary dependencies are available
The two dependencies you need to add to your pom.xml includes:

spring-boot-starter-jpa
spring-boot-starter-security
mysql-connector-java
You can get these dependencies in Maven Repository. You can also copy them from below:

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.15</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.1.4.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.1.6.RELEASE</version>
</dependency>
 

3. Create the User entity
You will also need to create a class that represents the User table in the database. This class should be the same name with the table created in MySQL.

Follow the steps below:

Step 1: Create the User class

Step 2: Annotate this class with the @Entity annotation

Step 3: Annotate the Id with the @Id annotation.

Step 4: Generate the Constructors as well as the getters and setters.

At the end, the the User class will be as shown below.

 

@Entity
public class User {
	
	@Id
	private Integer Id;
	private String username;
	private String password;
	
	//Constructor
	//Getters and Setters

}
 

 

4. Create the User Repository
Anytime you need to access data from a database, you would need to create a repository.

Follow the steps below to add a repository:

Step 1:  In the repositories package, create an interface, UserRepository.

Step 2: Annotate this class with the @Repository annotation

Step 3: Make this class extend jpaRepository.

Step 4: Specify the entity type and the primary key type

Step 5: Add a method definition for findByUsername. Ittakes a String parameter of username and returns a user object.

The content of the UserRepository is as shown below:

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
	
	User findByUsername(String username);

}
 

 

5. Configure AuthenticationProvider
We would still use the AppSecurityConfig file we created in the previous lesson. But this time, we would configure the provider. We need to create a method that returns type AuthentictionProvider. Inside this method;

Follow the steps below:

Step 1: Delete everything from the AppSecurityConfig class ( you can also comment them out if that’s what you prefer)

Step 2: Autowire UserDetailsService into this class (UserDetailsService is not yet created)

Step 3: Create a public method that returns AuthenticationProvider object. I name it authProvider

Step 4: Annotate this method with the @Bean annotation so it becomes a spring bean.

Step 5: Create a new DaoAuthenticationProvider object. I name it provider.

Step 6: Set the UserDetailsService  of this provider to the autowired UserDetailsService using setUserDetailsService

Step 7: Set the PasswordEncoder of the provider to NoOpPasswordEncoder.getInstance using setPasswordEncoder.

Step 8: Then return the DaoAuthenticationProvider object, provider.

At this point, the AppSecurityConfig class would be as shown below:

@Configuration
@EnableWebSecurity
public class AppSecurityConfig extends WebSecurityConfigurerAdapter {
	
	@Autowired
	private UserDetailsService userDetailsService;
	
	@Bean
	public AuthenticationProvider authProvider() {
		
		DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
		return provider;
	}	
}
 

6. Create the Service Layer (UserDetailsService)
Now we need to create a UserDetailsService interface. The UserDetailsService is used for retrieving user-related data from the repository. It contains on method called loadUserByUsername(). You can override this method to customize how a user is retrieved. In this example, we will retrieve a user and wrap it into a UserPrincipal object. This is what we’ll do.

Follow the steps below:

Step 1: Create a cIass, I named it MyUserDetailsService. In the New Class dialog box, add the UserDetails interface. This class provides a method called loadUserByUsername. It takes a String parameter of username.

Step 2: Annotate the class with the @Service annotation.

Step 3: Autowire the UserRepository into the UserDetailsService

Step 4:  Inside the loadUserByUsername method, create a User object. This object calls the repository findByUsername method, passing it the username parameter.

Step 5: Validate that if the user is null, then throw new UsernameNotFoundException

Step 6: Return new UserPrincipal object passing it the user you created in step 4.

At the end, the content of the service would be:

public class MyUserDetailsService implements UserDetailsService {
	
	@Autowired
	private UserRepository userRepository;

	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		
		User user = userRepository.findByUsername(username);
		if(user==null) {
			throw new UsernameNotFoundException("User not found!");
		}
		//so what do we return? So we would create a class that implements UserDetails
		
		return new UserPrincipal(user);

	}
}
 

 

7. Create a class the Implements UserDetails interface
We  will now create a class that implements the UseDetails interface. The UserDetails interface provides core user information as well as methods for managing users. Follow the steps below:

Step 1: Create a new class inside the models package . This class should implement the UserDetails interface. Choose the interface from the New Class dialog box. I call this class UserPrincipal. This is because Principal means the current user.

Step 2: Set the methods in this class to return true by default

Step 3: Add one more private field of type User to this class

Step 4: Generate a constructor that takes a User object as parameter

Step 5: Then modify the getPassword() and getUsername() methods to return user.getPassword() and user.getUsername() respectively.

Step 6: Also, modify the getAuthorities() method to return Authorities collection. This you can do using the line below:

return  Collections.singleton(new SimpleGrantedAuthority("USER"));
 

8. Final AppSecurityConfig File
For your final AppSecurityConfig, check that:

the UserDetailsService of the provider is set
the PasswordEncoder is set as well
that there is no error!
The final content of the AppSecurityConfig file would be as follows:

 

@Configuration
@EnableWebSecurity
public class AppSecurityConfig extends WebSecurityConfigurerAdapter {
	
	@Autowired
	private UserDetailsService userDetailsService;
	
	@Bean
	public AuthenticationProvider authProvider() {
		
		DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
		
		provider.setUserDetailsService(userDetailsService);
		
		provider.setPasswordEncoder(NoOpPasswordEncoder.getInstance());
		return provider;
	}	
}
 

9. Test the Application
At this point, you have completed the configuration. Congrats!

Launch the application. Visit the home page http://localhost:8080/home.

Try to login with the test user you added to the mySQL database. If you are able, to log in, great!. If not, watch the video lesson to learn why

 

10. Next Steps
Now we have been able to authenticate using data stored in a database. But some things are yet to be done. For instance, how do we create user details? Or maybe how do we customize the login form or use our own login form.
