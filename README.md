# aspdotnet-webapi-docker
This is a simple tutorial on how to Dockerize an ASP.NET Web API App

## Prequisites
### Docker
```
$ docker -v
Docker version 18.09.2, build 6247962
```

### Visual Studio for Mac version 8.0 or later

### .NET Core 3.0 SDK or later

### Git
```
$ git --version
```
### Account at https://hub.docker.com/

## Part 1: Setup Steps
### 1.1: Create a folder to hold the project
```
$ mkdir TodoApi
```
### 1.2: Navigate to that directory
```
$ cd TodoAPI
```

## Part 2: Create the ASP.NET Web API Hello World Application
### 2.1: Create web project using Visual Studio and Test the API Works

### 2.2: Add Entity Framwork Packages
```
$ dotnet add package Microsoft.EntityFrameworkCore.SqlServer
$ dotnet add package Microsoft.EntityFrameworkCore.InMemory
```

### 2.3 Add a model Class

```
namespace TodoApi.Models
{
    public class TodoItem
    {
        public long Id { get; set; }
        public string Name { get; set; }
        public bool IsComplete { get; set; }
    }
}
```

### 2.4 Add Database Context

```
using Microsoft.EntityFrameworkCore;

namespace TodoApi.Models
{
    public class TodoContext : DbContext
    {
        public TodoContext(DbContextOptions<TodoContext> options)
            : base(options)
        {
        }

        public DbSet<TodoItem> TodoItems { get; set; }
    }
}
```

### 2.5 Register the database context > Startup.cs
```
// Unused usings removed
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.EntityFrameworkCore;
using TodoApi.Models;

namespace TodoApi
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        public void ConfigureServices(IServiceCollection services)
        {
            services.AddDbContext<TodoContext>(opt =>
               opt.UseInMemoryDatabase("TodoList"));
            services.AddControllers();
        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseHttpsRedirection();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}
```

### 2.6 Scaffold the controller
```
$ dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
$ dotnet add package Microsoft.EntityFrameworkCore.Design
$ dotnet tool install --global dotnet-aspnet-codegenerator
$ dotnet aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers
```

### 2.7 Replace the PostTodoItem create method


### 2.X: Create the Dockerfile

## Part 3: Let's Build, Tag, Run our app locally
### 3.1: Build and tag our docker image
```
$ docker build -t aspdotnet-webapi-docker:latest .
```
```
...
Successfully built cf8265dd7d82
Successfully tagged aspdotnet-webapi-docker:latest
```

### 3.2: View Docker Images
```
$ docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
python_flask_docker   latest              cf8265dd7d82        19 seconds ago      921MB
python                2.7                 92c086fc9702        2 weeks ago         914MB
```

### 3.3: Run the build
```
$ docker run -d -p 5000:5000 python_flask_docker:latest
ad04385ef6
```

### 3.4: View Containers Running
```
$ docker ps
CONTAINER ID        IMAGE                        COMMAND             CREATED             STATUS              PORTS                    NAMES
ad04385ef6dc        python_flask_docker:latest   "python app.py"     9 seconds ago       Up 8 seconds        0.0.0.0:5000->5000/tcp   cranky_elion
```

### 3.5: View the Restful API locally using CURL, Browser or Postman
```
$ curl http://127.0.0.1:5000/
{
    "hello": "world"
}
```

## Part 4: Let's Push our Image to Docker Hub
### 4.1: Login to Docker Hub
```
$ docker login -u jrdalino
Password:
Login Succeeded
```

### 4.2: Let's re-tag the image with our username prefix
```
$ docker tag python_flask_docker jrdalino/python_flask_docker
```

### 4.3: Push the image to Docker Hub
```
$  docker push jrdalino/python_flask_docker

```

## Part 5: Let's stop and remove our local docker instance and pull it back from Docker Hub
### 5.1: Let's get the Container ID of docker container
```
$ docker ps
CONTAINER ID        IMAGE                        COMMAND             CREATED             STATUS              PORTS                    NAMES
ad04385ef6dc        python_flask_docker:latest   "python app.py"     3 minutes ago       Up 3 minutes        0.0.0.0:5000->5000/tcp   cranky_elion
```

### 5.2: Let's kill our container
```
$ docker kill ad04385ef6dc
ad04385ef6dc
```

### 5.3: Remove all docker images
```
$ docker system prune -a
```

### 5.4: Our docker image is gone
```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

### 5.5: Let's pull back our image from Docker Hub
```
$ docker pull jrdalino/python_flask_docker
...
Status: Downloaded newer image for jrdalino/python_flask_docker:latest
```

### 5.6: We should have our image back
```
$ docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
jrdalino/python_flask_docker   latest              cf8265dd7d82        8 minutes ago       921MB
```

### 5.7: Let's run it now
```
$ docker run -d -p 5000:5000 jrdalino/python_flask_docker:latest
71218cbf84422b53238416b69417135cb540f2b8c1f350b9d4f4b23ed1cc0ef5
```

### 5.8: It is alive again
```
$ docker ps
CONTAINER ID        IMAGE                                 COMMAND             CREATED             STATUS              PORTS                    NAMES
71218cbf8442        jrdalino/python_flask_docker:latest   "python app.py"     8 seconds ago       Up 7 seconds        0.0.0.0:5000->5000/tcp   gracious_tereshkova
```

### 5.9: View the Restful API locally using CURL, Browser or Postman
```
$ curl http://127.0.0.1:5000/
{
    "hello": "world"
}
```

## And we're done!
