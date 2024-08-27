## Serialization 
##### Serializers are used to convert complex data types, such as Django model instances, into Python data types that can be easily rendered into JSON, XML, or other content types.
##### Serializers also provide deserialization, allowing parsed data to be converted back into complex types after first validating the incoming data.
##### Serializers in Django are a part of the Django REST framework, a powerful and flexible toolkit for building Web APIs.

### Basic serializer model
##### model ```Todo``` present in ```models.py```
```python
class Todo(BaseModel):
    todo_title = models.CharField(max_length=100)
    todo_description = models.TextField()
    is_done = models.BooleanField(default=False) 
```

##### serializing model Todo in ```serializer.py``` in class ```TodoSerializer```

### Deserializing data when extracting from Database/model
##### Data should be deserialized in simple form like json before sending as the data extracted from Database or model is in form of ```queryset``` an object of the model
we can include and exclude fields we want to deserialize or send from the model object
using meta class by giving model and fields
```python
from rest_framework import serializers
from .models import Todo
import re
from django.template.defaultfilters import slugify

class TodoSerializer(serializers.ModelSerializer):
    # use meta class for simple ,automatic handling
    # use update function for custom logic and more control
    class Meta:
        model = Todo
        # fields = '__all__'  # deserialize all fields (all fields are returned )
        # fields = ['todo_title','todo_description']  # deserialize only selected fields
        exclude = ['created_at']  # exclude from deserialization
```


### Serialization of data using serializer

Before storing data it should be validated such that user have entered mandatory fields.
Here  ```todo_title``` and ```todo_description``` are mandatory feilds

###### Serializing functional based api view
###### functional based api view : 
- not manageable
- need to create multiple functions
- we will use class based api view

ex ```views.py```
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .serializer import TodoSerializer

@api_view(['POST'])
def post_todo(request):
    data = request.data
    serializer = TodoSerializer(data=data)
    
    # validate such that mandatory field are provided by user
    if serializer.is_valid():
        serializer.save()  # Save the valid data to the database
        return Response(serializer.data, status=status.HTTP_201_CREATED)
    else:
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### Adding Validation using Serializer
validate data before storing such that it is of correct format. In this case ```todo_title``` should not contain any special characters.
##### adding validate function inside Todoserializer ```serializer.py```
```python
def validate(self, validated_data):
        if validated_data.get('todo_title'):
            todo_title = validated_data['todo_title']
            regex = re.compile('[@_!#$%^&*()<>?/\|}{~:]')

            if len(todo_title) < 3:
                raise serializers.ValidationError('todo_tile should not be less than 3 characters')
            if regex.search(todo_title) is not None:
                raise serializers.ValidationError('todo_ title cannot contains special characters')
        return validated_data
```
#### we can also add specific validators for specific fields by adding ```_field_name``` to the validate function.
```python
def validate_todo_title(self, data):
        if data:
            todo_title = data
            regex = re.compile('[@_!#$%^&*()<>?/\|}{~:]')
            if len(todo_title) < 3:
                raise serializers.ValidationError('todo_tile should not be less than 3 characters')
            if regex.search(todo_title) is not None:
                raise serializers.ValidationError('todo_ title cannot contains special characters')
        return data
```

