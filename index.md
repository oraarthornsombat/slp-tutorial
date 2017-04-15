Preventing data duplication with HTTPResponse Redirects and Bootstrap Modals
=======================================================================

### Purpose of the tutorial
----------

Django provides a lot of built-in functionality through their generic class-based views. One functionality is handling of form validation errors. For example, if a user tries to submit data in a form for a model that duplicates the information contained in an existing model, the Django messaging framework can be used to display a quick and simple error message. However, the error message appears as an unpleasant-looking bulleted list and offers no quick solution to the user. 


<img src="images/django_messages.png" align="center">
*Django default rendering of form validation errors*

To enhance the user experience, we can utilize Bootstrap Modals and HTTPResponse Redirects. This tutorial will demonstrate how to detect data duplication, warn the user before he or she submits the form, and redirect the user to an Edit view if they attempt to create a new model with data duplication. The user will then easily be able to update the already existing model, instead of having to find it on their own.

### Start with a DeleteView
Consider a Book model with a basic DeleteView.  

* models.py  
```
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=50)
```

* views.py  
```
from django.views.generic.edit import DeleteView
from django.urls import reverse_lazy
from myapp import models

class DeleteBook(DeleteView):
    model = models.Book
    success_url = reverse_lazy('view_books')
    context_object_name = "book"
```  

* book_confirm_delete.html  
```
<form action="" method="post">
    {% csrf_token %}
    <p>Are you sure you want to delete "{{ book }}"?</p>
    <input type="submit" value="Confirm" />
</form>
```  

* book_lookup.html (or a similar file with the delete option)  
```
<html>
    <head></head>
    <body>
        <table>
            <tr>
                <th>Title</th>
                <th>Author</th>
                <th>Options</th>
            </tr>
            <tr>
                <td>{{ book.title }}</td>
                <td>{{ book.author }}</td>
                <td><a href="{% url 'delete_book' book.id %}">Delete</a></td>
            </tr>
        </table>
    </body>
</html>
```
### Switching to a modal for confirmation

1. Include Bootstrap in your project:  

* book_lookup.html  
```  
    <head>
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
        <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.2.0/jquery.min.js"></script>
        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
    </head>
```  

2. Add a modal to the body of the template:  
  
* book_lookup.html
```
<!--Modal-->
<div id="confirmDelete" name="confirmDelete" class="modal fade" role="dialog">
    <div class="modal-dialog">
        <!-- Modal content-->
        <div class="modal-content">
            <div class="modal-header">
                <button type="button" class="close" data-dismiss="modal">&times;</button>
                <h4 class="modal-title">Confirm Delete</h4>
            </div>
            <div class="modal-body">
                <p>Are you sure you want to delete this book?</p>
            </div>
            <div class="modal-footer">
                <form id="confirm_delete" action="" method="post">
                    {% csrf_token%} 
                    <button type="button" class="btn btn-danger" data-dismiss="modal">Cancel Delete</button>
                    <input type="submit" class="btn btn-success" value="Confirm Delete"/>
                </form>
            </div>
        </div>
    </div>
</div>
```  

The modal includes a title and a close button in the modal header, and a message to the user in the body. The form in the modal footer is what will send a POST request to the Django DeleteView to actually delete the object. However, right now the form has no action. JavaScript can be used to trigger the modal and set the delete action for the form.  

3. Alter the delete button in the html file:  

* book_lookup.html
```
<table>
    <tr>
        <th>Title</th>
        <th>Author</th>
        <th>Options</th>
    </tr>
    <tr>
        <td>{{ book.title }}</td>
        <td>{{ book.author }}</td>
        <td><a name="openConfirmDelete" id="openConfirmDelete" data-toggle="modal" data-target="#confirmDelete" onclick="confirmDelete('{% url 'delete_book' book %}')">Delete</a></td>
    </tr>
</table>
```  
By setting the data-target to the modal's ID, clicking to delete a book will toggle the modal (displaying it). In addition, setting the JavaScript function confirmDelete to be called when Delete is clicked will allow the action of the form in the modal footer to be set. Note that the JavaScript function takes the delete URL as an argument.  

4. Add the following JavaScript function to the html template:  

* book_lookup.html
```
<head>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.2.0/jquery.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
    <script type="text/javascript">
        function confirmDelete(action) {
            document.getElementById("confirm_delete").action = action;
        }
    </script>
</head>
```  

When a user clicks to delete, the action of the form in the modal will be set to the delete book URL and the modal will display. Because the form sends a POST request, the item will be deleted when the user clicks confirm.  
