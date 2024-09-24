---
layout: post
title: "A Step-by-Step Guide to Creating a Blog on ASP.NET Core"
date: 2024-09-24 00:00:00+0800
categories: [ASP.NET Core]
tags: [ASP.NET Core]
author: kayran
---

### Project Introduction 


In this project, a blog site was developed using the ASP.NET Core Web template. The main purpose of the project is to create a platform with user login and logout, where you can create and share blog posts and add comments. The blog site was created with the Web template instead of the MVC structure and database management was done using Entity Framework Core (EF Core). Since the web template was used, view files had to be added manually after the controller was created. Therefore, before the build process in Program.cs file, builder.Services AddControllersWithViews(); statement was added to make the controller and view structures ready for use in the project. In the project creation phase, the following command was used with the .NET command line tool:
```c#
dotnet new web -o BlogApp -f net8.0
``` 
With this command, an ASP.NET Core web application named BlogApp was created.

### Project Creation and Structuring 
{: data-toc-skip='' .mt-4 .mb-0 }

#### ASP.NET Core Project Setup 


After creating the project, the database infrastructure was prepared as the first step. In this process, Entity structures (table representations) of the database were created using EF Core and relationships between tables were defined. This relationship structure allows EF Core to understand which classes represent the tables in the database.
```c#
public class BlogContext : DbContext
{
    public DbSet<Post> Posts { get; set; }
    public DbSet<Tag> Tags { get; set; }
}
```
Also, the required database connection string has been added to the appsettings.Development.json file.


#### Seed Data Usage
{: data-toc-skip='' .mt-4 }

After the database is created, SeedData is added, which refers to sample data predefined in the database. SeedData adds data to the database before running the project. It is important that this data is organized according to the relationships between the tables. When SeedData is updated and applied to the project again. First the existing database is deleted with the following command:

```c#
dotnet ef database drop --force --context BlogContext
```
Afterwards, the project is run again and the SeedData is added to the database again.

### Project Architecture and Layers
{: data-toc-skip='' .mt-4 .mb-0 }

In the project, two subfolders were created inside the Data folder: Abstract and Concrete.
1. Abstract: This folder contains the interfaces.
2. Concrete: This folder contains the classes that we connect the interfaces to EF Core.

In the project, it is not a safe approach to use the BlogContext class directly, because it provides access not only to the required Post data, but also to other database entities such as _context.Tag or _context.User. Since this is undesirable, separate interfaces are defined for each table or entity to create a more secure and modular structure. In this way, only the needed data is accessed and database management is made more secure. For example, the IPostRepository interface manages only post data, while other entities are abstracted from this structure.
For example, an IPostRepository interface and the EfPostRepository class that implements this interface are defined as follows:
```c#
public interface IPostRepository
{
    IQueryable<Post> GetPosts();
}

public class EfPostRepository : IPostRepository
{
    private readonly BlogContext _context;

    public IQueryable<Post> GetPosts()
    {
        return _context.Posts;
    }
}
```
This structure provides a more flexible and manageable architecture by abstracting access to the database in the project. 
Also, these repositories have a scoped lifetime. Therefore, in the Program.cs file, before the application is compiled, this scope is specified by adding the following code:
```c#
builder.Services.AddScoped<IPostRepository, EfPostRepository>();
```
This creates a new repository instance for each HTTP request and ensures that dependencies within the application are properly managed. In addition, for pages that use the Razor View Engine in the project, a ViewImports file has been created to prevent specific namespaces or Razor directives from being defined separately in each view and to ensure that they apply globally. In the ViewImports.cshtml file, the definitions made using the @ sign are configured to be globally valid for all views. This allows us to use certain features across the entire project without adding them to each view file separately.

### Data Visualization and Bootstrap Integration
{: data-toc-skip='' .mt-4 .mb-0 }


The project uses Bootstrap for visualization and page layout. The following steps were followed to include Bootstrap in the project:

```c#
dotnet tool install Microsoft.web.librarymanager.cli -g
libman init -p cdnjs
libman install bootstrap@5.3.3 -d wwwroot/lib/bootstrap
```
colored tags are displayed on the detail page. This gives users a more visually meaningful experience.

To make Bootstrap and other static files available, ```app.UseStaticFiles();``` has been added to Program.cs. This allows static files (CSS, JS, img, etc.) in the project to be used by the server. In this way, Bootstrap features can be included and used in the project.

### Blog Functionality and Using ViewComponents
{: data-toc-skip='' .mt-4 .mb-0 }


## Blog posts and tag information are managed with PostViewModel in the project. This model allows the post and tag information to be displayed at the same time. In addition, the ViewComponent structure is used to make some parts of the blog page reusable. For example, ```vc:tags-menu</vc> and vc:new-posts</vc>``` components are defined to display the tag menu and new blog posts. These structures can be called from any page to display dynamic content.

## In the project, URL name redirection has been done in order to correctly redirect users to the lists of blog posts and tags. For this process, a custom route is defined using MapControllerRoute in Program.cs file. Instead of MapDefault, a route is configured as follows:
```c#
app.MapControllerRoute(
    name: "posts_details",
    pattern: "posts/details/{url}",
    defaults: new { controller = "Posts", action = "Details" }
);
```
Also, the defined pattern is used in links with the href attribute. The correct definition of patterns in the href is necessary to redirect users to the relevant post or tag page.
A similar URL redirection was done for tags and a page was created listing the blog posts belonging to each tag. In this way, users can access the blog posts belonging to that tag by clicking on the tags.
In the project, TinyMCE editor was integrated to enable users to create and edit blog posts. TinyMCE allows the user to create content in HTML format as a rich text editor (WYSIWYG). In order to provide this integration, TinyMCE's CDN connection was included in the project:

```c#
<script src="https://cdn.tiny.cloud/1/no-api-key/tinymce/5/tinymce.min.js" referrerpolicy="origin"></script>
```
Then, the relevant field is selected using the init function to initialize TinyMCE:
```c#
tinymce.init({
  selector: '#contentArea' // Area used for blog content
});
```
With this action, the rich text editor is enabled in the designated area. Through this editor, the user can create blog posts in HTML format. In order to render the content written with HTML tags correctly, this content has been edited on the view side with the ```@Html.Raw(@model.Content)``` statement. In this way, user-generated content is displayed securely and HTML tags are included in the content.
This integration has enabled the blog content to be organized in a more flexible and user-friendly way.

### Enum and Color Assignments
{: data-toc-skip='' .mt-4 .mb-0 }
## In the project, different colors are assigned for tags added to blog posts. For this purpose, a color is assigned to each tag using Enum structure and these colors are displayed on the detail page. This provides users with a more visually meaningful experience.


### Adding Comments and Using AJAX
{: data-toc-skip='' .mt-4 .mb-0 }
On the detail page of blog posts, users can add comments. AJAX was used to prevent refreshing the page while adding comments. Instead of scrolling to the bottom of the page after commenting, the new comment is immediately visible on the page. This feature was achieved by pulling data through JSON files and the comment field was organized using jQuery. In addition, ```@RenderSection(“Scripts”, required: false)``` was added to layout.cshtml to run JavaScript codes on the relevant pages.

### User Login and Registration Procedures 
{: data-toc-skip='' .mt-4 .mb-0 }

Within the scope of the project, Login and Register pages were created for users to log in to the site. Cookie Authentication was used for login. This method provides a secure login by storing the user's credentials on the server. 

Session management was used to manage the login and logout processes of the users. In cases where users log in, necessary arrangements have been made to prevent the login button from being displayed again in the Layout section. Also, logged in users are not redirected back to the login page via URL while the session is open. This control is done as follows:
```c#
@if (!User.Identity!.IsAuthenticated)
{
    <a href="/user/login”>Login"</a>
}
```
During the registration process, the user was prompted for the Confirm Password field and this field was used to check whether the password entered by the user was correct. The Compare feature was used to compare the password and password verification fields.
In addition, a relationship was established between user comments and the user. That is, a user who is not logged in cannot comment and is redirected to the login page. The logged in user's information is displayed in the comments.

### Admin and User Roles
{: data-toc-skip='' .mt-4 .mb-0 }

In the project, there is a difference in authorization between users and administrators. While admin can see all blog posts, normal users can only see the posts they have written. This authorization is provided with the [Authorize] tag on PostController.
Also, admins can make blog posts active/passive. This is done with admin control on the Edit page. In addition to blog posts, menu assignment can be made. That is, the category of the blog can be determined and the posts can be listed according to this category.

### Result
{: data-toc-skip='' .mt-4 .mb-0 }

In this article, we have covered the step-by-step process of developing a blog site using ASP.NET Core. The project includes many modern web development techniques, from database configuration to user management, from Bootstrap integration to adding comments without page refresh using AJAX. This process offers a more efficient development experience with the powerful tools offered by ASP.NET Core.

You can reach the project on link below;

<https://github.com/sedefnurkayran/MyBlogApp>