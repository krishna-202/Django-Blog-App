# Blog-Post

* create a Blog_App project and blog app.

* create a templates and static folder in blog app directory.

* create a base, home, about html pages.

* create a base template required for our website in base.html page.

* Add a bootstrap and js content to base.html page.

* Extend the base.html page content to other html pages.

* Create a view for home and about page in views.py file.

* Add a url mapping for home and about page in urls.py file.

* Create a css file in static folder and load the static files in html pages.

* Run the server and make sure evrything working fine.

* Run a migrations to cretae auth_user table.

* Create a super user using python manage.py createsuperuser

* Verify admin page is working or not.

* Create a Post model in models.py file and make migrations.

* Create a users app and create a register,login,logout views.

* For registration create a custom registration form inherting the inbuilt UserCreationForm.

* For login and logout import a inbuilt class based Login and Logout views.

* Install a crispy forms and mention crispy_forms in installed apps.

* Mention CRISPY_TEMPLATE_PACK='bootstrap4' in settings.py file.

* Add a LOGIN_REDIRECT_URL in settings.py file.

* Create a Profile model with one field as user which has one to one relationship with django User model and another field as profile pic.

* Media root and Media url settings are added in settings.py file.

* Profile view is added in views.py file with login decorator.

* Note: We will access image based on URL(user.profile.image.url).

* MEDIA_ROOT (media folder) is the directory where the uploaded files are stored. For performance reason we store the images in file system not on database.

* MEDIA_URL is te way how we access the media files through browser.For ex: /media/profile_pic/default.jpg.

* Through user.profile.image.url we can access image in profile.html.

```
{% block content %}
  <div class="content-section">
  <div class="media">
    <img class="rounded-circle account-img" src="{{ user.profile.image.url }}">
    <div class="media-body">
      <h2 class="account-heading">{{ user.username }}</h2>
      <p class="text-secondary">{{ user.email }}</p>
    </div>
  </div>
  <!-- FORM HERE -->
  </div>
{% endblock %}
```

* We have add media root to url patterns only when it is in DEBUG mode.

```
if settings.DEBUG:
    urlpatterns+=static(settings.MEDIA_URL,document_root=settings.MEDIA_ROOT)
```

* For every new user created they should automatically get user profile which will include default profile pic

* Inside users app create signals.py file.

* we should get a post_save signal when user gets created.

* User model is the sender and also we need receiver to perform task.
```
from django.dispatch import receiver
```

* For creating profile
```
@receiver(post_save,sender=User)
def create_profile(sender,instance,created,**kwargs):
    if created:
        Profile.objects.create(user=instance)
```
* For saving the profile
```

@receiver(post_save,sender=User)
def save_profile(sender,instance,**kwargs):
    instance.profile.save()
```

* In apps.py file create a ready method
```
def ready(self):
        import users.signals
```

* Create a ListView and Detail View for the post model.

* By default list view expects a app_name/post_list.html template,post_list as context object name and detail view expects a app_name/post_detail.html template, post or object as context object name.

* Add CreateView in views.py file and author will be set to current logged in user. This is done by form valid method.
```
def form_valid(self,form):
  form.instance.author=self.request.user
  return super().form_valid(form)
```
  
* Inherit LoginRequiredMixins for CreateView which indicates one should login for creating post.

* Add PostUpdateView with LoginRequiredMixins inherited and also inherit userPassesTestMixin with test_func method which makes sure that only author of respected post can edit.
```
def test_func(self):
  post=self.get_object()
  if self.request.user == post.author:
    return True
  return False
```

* Add a PostDeleteView with LoginRequiredMixin,UserPassesTestMixin and add a success_url once post gets deleted.

## Paginator

* from django.core.paginator import Paginator
```
posts=['1','2','3','4','5']
p=Paginator(posts,2)
p.num_pages
o/p 3
```
* In above code (posts,2) 2 indicates 2 posts per page..so 3 will be number of pages.

```
for page in p.page_range:
  print(page)
  
p1=p.page(1)
p1.number   o/p is 1
```
* To see ow many posts are there in that page
```
p1.object_list
o/p ['1','2']

p1.has_previous()
o/p false

p.has_next()
o/p true

p1.next_page_number()
o/p is 2

```

* Add paginate_by attribute in PostListView.

* We can navigate to different page number. Url should be localhost:8000/?page=n where n is the page number you want to navigate.

* Go to home.html and create a template for page numbers.

* If page_obj has previous page then build a navigation page for very first page and previous page.
```
{% if is_paginated %}
  {% if page_obj.has_previous %}
    <a class="btn btn-outline-info mb-4" href="?page=1">First</a>
    <a class="btn btn-outline-info mb-4" href="?page={{ page_obj.previous_page_number }}">Previous</a>
  {% endif %}
  ```
  
* Add 3 preceeding and follwing navigation pages to the current page.
```
% for num in page_obj.paginator.page_range %}
    {% if page_obj.number == num %}
      <a class="btn btn-info mb-4" href="?page={{ num }}">{{ num }}</a>
        {% elif num > page_obj.number|add:'-3' and num < page_obj.number|add:'3' %}
          <a class="btn btn-outline-info mb-4" href="?page={{ num }}">{{ num }}</a>
    {% endif %}
  {% endfor %}
  ```
  
  * If page_obj has next page then build a navigation page for very last page and next page.
  
  * Build a UserPostListView in views.py file which lists out all the posts with respect to users.
 ```
 class UserPostListView(ListView):
    model=Post
    template_name='blog/user_posts.html'
    context_object_name='posts'
    paginate_by=5

    def get_queryset(self):
        user=get_object_or_404(User,username=self.kwargs.get('username'))
        return Post.objects.filter(author=user).order_by('-date_posted')
```

* Create a template and url mapping for UserPostListView.

* 



