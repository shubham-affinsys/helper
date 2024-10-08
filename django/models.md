### How to create models in models.py
```python
from django.db import models
class Car(models.Model):
    car_name = models.CharField(max_length=200)
    speed = models.IntegerField(default=100)

    def __str__(self) -> str:
        return self.car_name +" "+str(self.speed)

#store data in an order -->
class Department(models.Model):
    department = models.CharField(max_length=100)

    def __str__(self):
        return self.department

    class Meta:
        ordering = ['department'] # data will be stored in a specific order department name in this case
```

### MIGRATION

when we make changes in models we need to make migrations only then changes will take effect

1. ```python3 manage.py makemigrations```
2. ```python3 manage.py migrate``` 

### CREATE
 
create car obj
1. ```python
    car = Car(car_name="Nexon",speed=110) # need to save using car.save()
    ```
   
2. ```python
    car = Car.objects.create(car_name="Nexon",speed=110)  # auto save
    ```
3. ```python
    car_obj = {'car_name':'alto','speed':120}
    car = Car.objects.create(**car_obj)
   ```

### UPDATE

update a created object 

```python

    Car.objects.filter(id=1).update(car_name="new_car")

```

>1

### DELETE

```python
Car.objects.all().delete()
Car.objects.filter(id=1).delete()
```


### READ:
```python
cars = Car.objects.all()
```

<QuerySet [<Car: Car object (1)>, <Car: Car object (2)>, <Car: Car object (3)>, <Car: Car object (4)>]>

```python
car = Car.objects.get('car_name'='Nexon')
```
>car
>>Nexon 120

if attribute is an object ex student has attirubute department that is foregn key or studnet_id

```python
Student.objects.get(student_name'"shubh").department
```

it  will give department object while
```python
Student.objects.get(student_name'"shubh").department.department
```

It will give departmnet name
```python
s = Student.objects.get(student_name="shubh")
```

> s.student_id
>><StudentID: STU-0001>

> s.student_id.student_id
>>'STU-0001'

```python
queryset = Student.objects.get(student_name="shubh") # will raise an exception if the data does notexists
queryset = Student.objects.filter(student_age__gte=20)
queryset = Student.objects.exclude(student_name__icontains="s")
queryset = Student.objects.order_by("-student_age)  # -ve for desc
queryset = Student.objects.order_by('student_id').distinct('student_id')
queryset = Student.object.value_list('id','student')   # fetch only specific feilds

queryset.count() # give size of results
queryset.values()  # makes the reuslt as dict
```

### AGGREGATE
function work on single column ex student_age
```python
queryset = Student.objects.aggregate(Avg('student_age'))
queryset = Student.objects.aggregate(Max('student_age'))
queryset = Student.objects.aggregate(Min('student_age'))
queryset = Student.objects.aggregate(Sum('student_age'))
```

### ANNOTATE
works on multiple cloumns ex count of students that have same age and all ages
```python
# count of student in each student age group
queryset = Student.objects.values('student_age').annotate(Count('student_age'))
queryset = Student.objects.values('department').annotate(Count('department'))

#multi annotate
queryset = Student.objects.annotate(Count('department'),Count('student_age')) # objects
queryset = Student.objects.values('department','student_age').annotate(Count('department'),Count('student_age')) # values dict
```

### LOOKUPS
```python
#get query in a specific sort order --> descending order
queryset = Recipe.objects.all().order_by('-recipe_view_count')

#fetch limited results --> fetch most viewed 10 results
queryset = Recipe.objects.all().order_by('-recipe_view_count')[0:10]

#fetch only recipes with view count more than or less 50
queryset = Recipe.objects.filter(recipe_view_count=50) # less than 50
queryset = Recipe.objects.filter(recipe_view_count__gte=50) # greater than than 50
```

#### custom lookups based on fields/attribute of model :
```python
feild__custom_para = "val"
# ex
Student.objects.filter(department__department=="CIVIL")

#can also merge multiple in one statement
Student.objects.filter(department__department__icontains=="C")  # will include CIVIl and CSE department
```

### READ lookup
```
__gt : greater than
__gte : greater than equal to
__lte : less than equal to
__exact : exact match
__iexact : case insensitive exact match
__contains : check for substring
__icontains : case insensitive substring match
__in=[val1,val2,val3] : if value is in given list
__startswith : check if the value starts with a string
__istartswith
__endswith
__iendswith
```

### DATE LOOKUPS
```
__date = date.time(2024,8,20)
__year=2024
__month=8
__day=8
__week
__week_day=1  # monday
__quarter=3
__time = date.time(14,30)
__hour=14
__minute=30
__second=0
```

### BOOLEAN and NULL
```
__regex = r"^[A-Za-z0-9]+$"
__iregex
__range = (start_val,end_val)
```
### F expresion lookup
compare one field to other
```
filed__gte = F('other_field)
```

### JSON filed lookup
```
__has_key = 'key_name'
__has_keys = ['key1','key2']
__has_any_keys = ['key1','key2']
```

### ADD FAKE DATA USING FAKER
```pip install faker```

create ```file seed.py```

```python
from faker import Faker
import random
from .models import *

fake = Faker()

def seed_db(n=10) -> None:
    try:
        for _ in range(n):
            department_objs = Department.objects.all()
            random_idx = random.randint(0, len(department_objs)-1)
            department = department_objs[random_idx]

            student_id = f'STU-0{random.randint(100, 999)}'
            student_name = fake.name()
            student_email = fake.email()
            student_age = random.randint(18, 30)
            student_address = fake.address()

            student_id_obj = StudentID.objects.create(student_id=student_id)
            student_obj = Student.objects.create(
                department=department,
                student_id=student_id_obj,
                student_name=student_name,
                student_email=student_email,
                student_age=student_age,
                student_address=student_address
            )
    except Exception as e:
        print(e)
```

Add following code  in file ```views.py``` in vege
```python

from django.shortcuts import render, redirect
from .models import Recipe
from django.contrib.auth.models import User
from django.contrib import messages
from django.contrib.auth import authenticate, login,logout
from django.contrib.auth.decorators import login_required

@login_required(login_url="/login/")
def recipes(request):

    if request.method == "POST":
        data = request.POST
        recipe_image = request.FILES.get('recipe_image')
        recipe_name = data.get('recipe_name')
        recipe_description = data.get('recipe_description')

        recipe = Recipe.objects.create(
            recipe_image=recipe_image,
            recipe_name=recipe_name,
            recipe_description=recipe_description
        )
        print("recipe stored successfully =====>", recipe.id)
        return redirect("/vege/recipes/")

    queryset = Recipe.objects.all().order_by('-recipe_view_count') # - for sorting in desc

    if request.GET.get('search'):
        queryset = queryset.filter(recipe_name__icontains=request.GET.get('search'))

    context = {'recipes': queryset}
    return render(request, 'recipes.html', context=context)


def delete_recipe(request, id):
    queryset = Recipe.objects.get(id=id)
    queryset.delete()
    print("recipe deleted ===>", id)
    return redirect('/vege/recipes/')


def update_recipe(request, id):
    queryset = Recipe.objects.get(id=id)
    if request.method == "POST":
        data = request.POST
        recipe_image = request.FILES.get('recipe_image')
        queryset.recipe_name = data.get('recipe_name')
        queryset.recipe_description = data.get('recipe_description')

        if recipe_image:
            queryset.recipe_image = recipe_image

        # Recipe.objects.filter(id=id).update(
        #     recipe_image=recipe_image,
        #     recipe_name=recipe_name,
        #     recipe_description=recipe_description
        # )

        queryset.save()
        print("recipe updated successfully =====>", id)
        return redirect("vege/recipes/")
    context = {'recipe': queryset}
    return render(request, 'update_recipe.html', context)
```



### Creating a base model

#### default django models does not have uuid 
```models.py```
```python
class BaseModel(models.Model):
    uid = models.UUIDField(primary_key=True, editable=False, default=uuid.uuid4())
    created_at = models.DateField(auto_now=True)
    updated_at = models.DateField(auto_now_add=True) 

    class Meta:
        abstract = True  # so that we can use it as a class
```


### ORM in Django 
#### creating relation bw models
```models.py```
```python

class StudentID(models.Model):
    student_id = models.CharField(max_length=100)

    def __str__(self):
        return self.student_id

class Student(models.Model):
    department = models.ForeignKey(Department, related_name='depart', on_delete=models.CASCADE)
    student_id = models.OneToOneField(StudentID, related_name='studentid', on_delete=models.CASCADE)
    student_name = models.CharField(max_length=100)
    student_email = models.EmailField(unique=True)
    student_age = models.IntegerField(default=18)
    student_address = models.TextField()

    def __str__(self):
        return self.student_name

    class Meta:
        ordering = ['student_name']
        verbose_name = "student"


class Subject(models.Model):
    subject_name = models.CharField(max_length=30)

    def __str__(self):
        return self.subject_name

class SubjectMarks(models.Model):
    student = models.ForeignKey(Student, related_name='studentmarks', on_delete=models.CASCADE)
    subject = models.ForeignKey(Subject, on_delete=models.CASCADE)
    marks = models.IntegerField()

    def __str__(self):
        return f'{self.student.student_name} {self.subject.subject_name}'

    class Meta:
        unique_together = ['student', #
                           'subject']  # every marks for subject will be unique and new subject of same name will not be created


```


### Serializing model data
 this model todo present in ```models.py``` will be serialized in ```serializer.py```
```pyhton
class Todo(BaseModel):
    todo_title = models.CharField(max_length=100)
    todo_description = models.TextField()
    is_done = models.BooleanField(default=False) 

```