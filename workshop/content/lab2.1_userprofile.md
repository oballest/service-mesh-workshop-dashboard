# Building a Microservice

<blockquote>
<i class="fa fa-desktop"></i>
In the browser, navigate to the 'Profile' section in the header.
</blockquote>

<p><i class="fa fa-info-circle"></i> If you lost the URL, you can retrieve it via:</p>

```execute
echo $GATEWAY_URL
```

<br>

You should see the following:

<img src="images/app-unknownuser.png" width="1024"><br/>
 *Unknown Profile Page*

The UI shows an unknown user and that's because there's no profile service for your application.  You are going to build a new microservice for user profiles and add this to your service mesh.

## Application Code

Your new application is written in Java, whereas the other backend components such as 'app-ui' and 'boards' are written in NodeJS.  One of the advantages of Istio is that it is agnostic to the programming languages of the running microservices.


<blockquote>
<i class="fa fa-terminal"></i>
Take a look at the "UserProfile" class in your repository:
</blockquote>

```execute
cat ./code/userprofile/src/main/java/org/microservices/demo/json/UserProfile.java | grep "public UserProfile(String" -A 7
```

Output:
```java
    public UserProfile(String id, String firstname, String lastname, String aboutme) {
        this.id = id;
        this.firstName = firstname;
        this.lastName = lastname;
        this.aboutMe = aboutme;
        this.createdAt = Calendar.getInstance().getTime();
    }
```

This class encapsulates information about the user such as the first and last name.

<br>

Your application also exposes a REST API to interact with the service.

<blockquote>
<i class="fa fa-terminal"></i>
Next, take a look at the "UserProfileService" class:
</blockquote>

```execute
cat ./code/userprofile/src/main/java/org/microservices/demo/service/UserProfileService.java | grep "UserProfile getProfile(" -B 5
```

Output:
```java
    /**
     * return a specific profile
     * @param id
     * @return the specified profile
     */
    UserProfile getProfile(@NotBlank String id);
```

This interface includes the REST methods for getting and setting user profile information.

<br>