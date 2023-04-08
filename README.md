# Django app to CRUD users using graphQL api

### Installation and setup

1. Project Setup
```
django-admin startproject graphql_django_project
cd graphql_django_project
python manage.py startapp users
```

2. Install Graphene-Django
```
pip install graphene-django

```

3. Configure the Django project
In graphql_django_project/settings.py, add the following lines:

```
INSTALLED_APPS = [
    # ...
    'graphene_django',
    'users',
]
```

4. Create a User model
In users/models.py, create a simple User model:


```
from django.db import models

class User(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)

```

5. Create GraphQL schema for the User model
Create a new file users/schema.py and define the GraphQL schema for the User model:

```
import graphene
from graphene_django import DjangoObjectType
from .models import User

class UserType(DjangoObjectType):
    class Meta:
        model = User
        fields = ("id", "first_name", "last_name", "email")

class Query(graphene.ObjectType):
    users = graphene.List(UserType)

    def resolve_users(self, info, **kwargs):
        return User.objects.all()

class CreateUserMutation(graphene.Mutation):
    user = graphene.Field(UserType)

    class Arguments:
        first_name = graphene.String(required=True)
        last_name = graphene.String(required=True)
        email = graphene.String(required=True)

    def mutate(self, info, first_name, last_name, email):
        user = User(first_name=first_name, last_name=last_name, email=email)
        user.save()
        return CreateUserMutation(user=user)

class UpdateUserMutation(graphene.Mutation):
    user = graphene.Field(UserType)

    class Arguments:
        id = graphene.ID(required=True)
        first_name = graphene.String()
        last_name = graphene.String()
        email = graphene.String()

    def mutate(self, info, id, first_name=None, last_name=None, email=None):
        user = User.objects.get(id=id)
        if first_name is not None:
            user.first_name = first_name
        if last_name is not None:
            user.last_name = last_name
        if email is not None:
            user.email = email
        user.save()
        return UpdateUserMutation(user=user)

class DeleteUserMutation(graphene.Mutation):
    success = graphene.Boolean()

    class Arguments:
        id = graphene.ID(required=True)

    def mutate(self, info, id):
        user = User.objects.get(id=id)
        user.delete()
        return DeleteUserMutation(success=True)

class Mutation(graphene.ObjectType):
    create_user = CreateUserMutation.Field()
    update_user = UpdateUserMutation.Field()
    delete_user = DeleteUserMutation.Field()

```

6. Create the project-level schema
Create a new file graphql_django_project/schema.py and define the project-level GraphQL schema:

```
import graphene
import users.schema

class Query(users.schema.Query, graphene.ObjectType):
    pass

class Mutation(users.schema.Mutation, graphene.ObjectType):
    pass

schema = graphene.Schema(query=Query, mutation=Mutation)
```

8. Configure the GraphQL endpoint
In graphql_django_project/urls.py, add the following lines to configure the GraphQL endpoint:

```
from django.urls import path
from graphene_django.views import GraphQLView
from .schema import schema

urlpatterns = [
    # ...
    path("graphql/", GraphQLView.as_view(graphiql=True, schema=schema)),
]
```

9. Now, run the following commands to apply the migrations and start the development server:

```
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```
You should now have a working GraphQL API for the User model. You can access the GraphiQL interface at http://127.0.0.1:8000/graphql/ and test the API by sending queries and mutations.

Here are some example queries and mutations:

- Create a user:

```
mutation {
  createUser(
    firstName: "John",
    lastName: "Doe",
    email: "john.doe@example.com"
  ) {
    user {
      id
      firstName
      lastName
      email
    }
  }
}
```

- List all users:

```
query {
  users {
    id
    firstName
    lastName
    email
  }
}

```

- Update a user:

```
mutation {
  updateUser(id: 1, firstName: "Jane", lastName: "Doe") {
    user {
      id
      firstName
      lastName
      email
    }
  }
}
```

- Delete a user:

```
mutation {
  deleteUser(id: 1) {
    success
  }
}
```