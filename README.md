# App-Builder-Project
<h1>Introduction</h1>
<p>During my time at the Tech Academy we were tasked with working on two different projects at<br>
different point in time while we were there. Our first project was a Python Django application.<br>
This project tasked us with creating our own individual interactive websites for managing<br>
one's collections of things related to various hobbies. Later on in the project we were also<br>
tasked with including an API as well. For my interactive website I chose to theme it around<br>
the popular auto-battler Teamfight Tactics or better known as TFT. You're able to peruse<br>
the various main and mid sets, learn about them, and create your own favorite or least<br>
favorite main and mid set. This project was a two week sprint. When it got to the point where<br>
we use an API, I unfortunately could not get access to the API Riot Games has for TFT<br>
within our two week time constraint. At that point I went and settled for a Magic the Gathering<br>
API.</p>
<br>
<h2>Stories</h2>
<ul>
  <li>Creating the App</li>
  <li>Creating the Model</li>
  <li>Displaying the Database Items</li>
  <li>Details Page</li>
  <li>Edit and Delete Functions</li>
  <li>Connecting to the API</li>
  <li>Parsing through JSON</li>
  <li>Front End Improvements</li>
</ul>
<br>
<h3>Creating the App</h3>
<p>The first story was creating the basic application and registering it. I created it <br>
using manage.py startapp then registered it within the settings.py. Afterwards I created<br>
the base and home templates within a new templates folder. Within the views the home page<br>
needed to be rendered, so I added the function to do that.</p>
```
<!--This is the home page to render -->
{% extends "tft_base.html" %}

{% block title %}The Teamfight tactics tier list{% endblock %}
{% block content %}
    <body class="homeBg_img">
    <h1 class="welcome font-weight-bold">Welcome to the Teamfight Tactics tier list!</h1>
    <hr>
    <div class="text-center">
        <h5 class="font-weight-bold">Explore all the TFT sets and decide your favorite and least favorite!</h5>
        <button type="button" class="btn btn-outline-dark btn-lg"><a href="{% url 'set1' %}">Set 1</a></button>
        <button type="button" class="btn btn-outline-dark btn-lg"><a href="{% url 'set2' %}">Set 2</a></button>
        <button type="button" class="btn btn-outline-dark btn-lg"><a href="{% url 'set3' %}">Set 3</a></button>
        <button type="button" class="btn btn-outline-dark btn-lg"><a href="{% url 'set4' %}">Set 4</a></button>
        <button type="button" class="btn btn-outline-dark btn-lg"><a href="{% url 'set5' %}">Set 5</a></button>
        <button type="button" class="btn btn-outline-dark btn-lg"><a href="{% url 'set6' %}">Set 6</a></button>
        <button type="button" class="btn btn-outline-dark btn-lg"><a href="{% url 'set7' %}">Set 7</a></button>
        <button type="button" class="btn btn-outline-dark btn-lg"><a href="{% url 'set8' %}">Set 8</a></button>

    </div>
    </hr>
    </body>

{% endblock %}
```
<br>
```
urlpatterns = [
    path('', views.tft_home, name='tft_index'),
    ]
```

<br>
<h3>Creating the Model</h3>
<p>I needed to next create the model for the databse. The model included two classes for FavoriteSet and<br>
LeastFavoriteSet. These both had input fields and set choices for the user. I added it to the views so<br>
it could render and added a template to my app folder for creating a new item. Right below is the model.<br></p>
```
from django.db import models

SET_CHOICES = {
    ('Set 1', 'Set 1'),
    ('Set 2', 'Set 2'),
    ('Set 3', 'Set 3'),
    ('Set 4', 'Set 4'),
    ('Set 5', 'Set 5'),
    ('Set 6', 'Set 6'),
    ('Set 7', 'Set 7'),
    ('Set 8', 'Set 8'),
}

MIDSET_CHOICES = {
    ('Set 1.5', 'Set 1.5'),
    ('Set 2.5', 'Set 2.5'),
    ('Set 3.5', 'Set 3.5'),
    ('Set 4.5', 'Set 4.5'),
    ('Set 5.5', 'Set 5.5'),
    ('Set 6.5', 'Set 6.5'),
    ('Set 7.5', 'Set 7.5'),
    ('Set 8.5', 'Set 8.5'),
}

class FavoriteSet(models.Model):
    title = models.CharField(max_length=60, default="", blank=True, null=False)
    set = models.CharField(max_length=60, choices=SET_CHOICES)
    midSet = models.CharField(max_length=60, choices=MIDSET_CHOICES)
    reason = models.CharField(max_length=300, default="", blank=True)

    objects = models.Manager()

    def __str__(self):
        return self.title



class LeastFavoriteSet(models.Model):
    title = models.CharField(max_length=60, default="", blank=True, null=False)
    set = models.CharField(max_length=60, choices=SET_CHOICES)
    midSet = models.CharField(max_length=60, choices=MIDSET_CHOICES)
    reason = models.CharField(max_length=300, default="", blank=True)

    objects = models.Manager()

    def __str__(self):
        return self.title
```
<br>
<p>The below function is what allows you to create your favorite set. If the method is POST and<br>
and the form is valid it saves it and returns you to the stored sets. It can then render the entries.<br>
There is an identical one for Least Favorite</p>
```
def create_favorite(request):
    form = CreateSetForm2(data=request.POST or None)
    if request.method == 'POST':
        if form.is_valid():
            form.save()
            return redirect('stored_sets')
    content = {'form': form}
    return render(request, 'Tft/favorite_set.html', content)
```
<br>
<h3>Displaying the Database Items</h3>
<p>The next task was displaying the information from the database to a page. That meant<br>
creating a new HTML page and link it from my home page. I added a function that fetches all the<br>
items from the database and sends them to the template. Below is the where the set entries got displayed.<br>
There is an identical version of this same code for Least Favorite sets as well.</p>
```
{% extends 'tft_base.html' %}
    {% block title %}Favorite Sets{% endblock %}
    {% block content %}
    <!--The entries for all the FAVORITE sets -->
    <body class="set1Bg_img">
    <h2>YOUR FAVORITE SETS</h2>

    <div class="row justify-content-center">
    <div class="col-auto">
    <table class="table table-hover table-striped table-responsive table-hover" id="set-table">

        <tr id="headerRow">
            <th>Title</th>
            <th>Set</th>
            <th>Mid Set</th>
            <th>Reason</th>
            <th></th>
        </tr>
        <!--This grabs all the entries and places them into a table-->
        {% for s in entry %}
        <tr>
            <td class="center"><a href="{% url 'fave_details_page' s.id%}">{{ s.title }}</a></td>
            <td>{{ s.set }}</td>
            <td>{{ s.midSet }}</td>
            <td>{{ s.reason }}</td>
            <td>
                <a class="btn btn-info edit-btn" href="{% url 'fave_set_edit' s.id %}" role="button">Edit Set</a>
            </td>
        </tr>
        {% endfor %}
    </table>
    </div>
    </div>
    </body>

    {% endblock %}
```
<br>
<p>This is where you could access your Favorite and Least Favorite sets</p>
```
<div class="fave_and_least">
        <button type="button" class="btn btn-light btn-lg"><a href="{% url 'favorite' %}">Add Favorite Set</a></button>
        <button type="button" class="btn btn-light btn-lg"><a href="{% url 'least' %}">Add Least Favorite Set</a></button>
        <button type="button" class="btn btn-light btn-lg"><a href="{% url 'stored_sets' %}">Your Favorite sets</a></button>
        <button type="button" class="btn btn-light btn-lg"><a href="{% url 'stored_sets2' %}">Your Least Favorite sets</a></button>
    </div>
```
<br>
<p>These two functions is what shows you the sets you have created. It grabs all the objects in the database,<br>
places it into a variable called content and renders it. There is an identical one for Least Favorite</p>
```
def stored_sets(request):
    #This is for FAVORITE sets
    entry = FavoriteSet.objects.all()
    content = {'entry': entry}
    return render(request, 'Tft/stored_set_entries.html', content)
    
def faveList(request):
    item = FavoriteSet.objects.all()
    content = {"item": item}
    return render(request,'Tft/stored_set_entries.html', content)
```
<br>
<h3>Details Page</h3>
<p>After getting the favorite and least favorite sets to display on their own page, the next<br>
thing to do was create a details page. The details page will hold the details of the one specific<br>
entry you want to look at. There is an identical version of the code below for the Favorite set.<br></p>
```
<div id="select_menu">
                {% for s in entry2 %}
                {{ s.title }}<br>{{ s.set }}<br>{{ s.midSet }}<br>{{ s.reason }}
                {% endfor %}
            </div>
```
<br>
<p>This is the function that will take you to the details page. It grabs the primary key of the entry<br>
you are asking for and filters the objects to just that one places it in content and renders it. There<br>
is an identical version for Least Favorite</p>
```
def stored_sets_details(request, pk):
    #This is for FAVORITE sets
    pk = int(pk)
    entry = FavoriteSet.objects.filter(pk=pk)
    content = {'entry': entry}
    return render(request, 'Tft/fave_details.html', content)
```
<br>
<h3>Edit and Delete Functions</h3>
<p>The last thing to implement for the details page is the Edit and Delete functions. This one<br>
was a challenge for me at first as nothing was permanently deleting from the databse. Originally<br>
if you edited an entry, it would edit a different entry as well. There also needed to be a page<br>
that would confirm if you wanted to delete the entry or not. Below is the delete function.<br>
Like the details function it would grab the primary key, confirm that the method was POST<br>
then delete the item. There is an identical one for Least Favorite</p>
```
def fave_delete(request, pk):
    pk = int(pk)
    item = get_object_or_404(FavoriteSet, pk=pk)
    if request.method == "POST":
        item.delete()
        return redirect('fave_and_least')
    context = {"item": item,}
    return render(request, "Tft/confirmDelete.html", context)
```
<br>
<p>The edit function required a little bit more code as it needed to call the CreateSetForm<br>
function as well. Like the other functions it would grab the Primary Key check for POST and validity<br>
and save the entry and redirect you. There is an identical version of this code for Favorite.</p>
```
def leastFaveEdit(request, pk):
    pk = int(pk)
    entry = get_object_or_404(LeastFavoriteSet, pk=pk)
    form = CreateSetForm(data=request.POST or None, instance=entry)
    if request.method == 'POST':
        if form.is_valid():
            form = form.save(commit=False)
            form.save()
            return redirect('stored_sets2')
    content = {'form': form, 'entry': entry}
    return render(request, 'Tft/set_edit2.html', content)
```
<br>
<h3>Connecting to the API</h3>
<p>The next two stories were all about learning to use API's. I chose a Magic the Gathering API<br>
because the TFT API from Riot Games would take too long to be able to access. Once I was<br>
connected to the database it would print a small text snippet on the screen. The next<br>
story had all of the bulk to it.</p>
<br>
<h3>Parsing through JSON</h3>
<p>For this story I was required to get elements out of my JSON response and display them on<br>
the page. For mine I decided that it would interesting to make a search bar instead. The<br>
search bar would look up any magic card you input and it would grab all the cards with<br>
that name and give you the colors and cmc of the cards. It was my hardest story on the project.<br>
the Code below is the search method. This is what allowed you to use the search button</p>
```
def mtg_search(request):
    if request.method == "POST":
        name = request.POST.get('mtg_query')
        url = f"https://api.magicthegathering.io/v1/cards?name={name}"
        response = requests.get(url)
        data = response.json()
        context = {'cards': data.get('cards')}
        return render(request, 'Tft/mtg_search.html', context)
```
<br>
<p>The Api method would then loop over all the cards in the response and extract<br>
their names and cmc. It will then pass the list of cards that fit the match to the<br>
template context.</p>
```
def tft_api(request):
    if request.method == "POST":
        name2 = request.POST['mtg_query']
        url = "https://api.magicthegathering.io/v1/cards"
        search = f"{url}name={name2}"
        response = requests.get(search)

        if response.status_code == 200:
            data = response.json()

            if data.get("cards"):
                cards = []
                for card in data["cards"]:
                    name = card.get("name", "")
                    cmc = card.get("cmc", "")
                    cards.append({"name": name, "cmc": cmc})
                content = {"cards": cards}
            else:
                content = {"error": "No cards found."}
        else:
            content = {"error": f"Request failed with status code {response.status_code}."}

        return render(request, 'Tft/tft_API.html', content)
    else:
        return render(request, 'Tft/tft_API.html', {})
```
<br>
<h3>Front End Improvements</h3>
<p>Last but not least was Front End Improvements. The webpage needed a lot of<br>
help to look presentable. I won't post all of it since it is a lot, but here are some<br>
examples. for the first one I created a static background that wouldn't move as the<br>
user scrolled.<br></p>
```
.set1Bg_img {
    background: url("/static/images/app-images/tfthome.png") no-repeat center center fixed;
    background-size: cover;
}
```
<br>
<p>The footer had quite a bit of styling to it. This is only a bit of it, but there<br>
is more for left side along with certain elements as well.</p>
```
#block-footer {
    clear: both;
    display: flex;
    width: 100%;
    -moz-box-sizing: border-box;
    margin: 0 auto;
    background-color: darkslateblue;
    position: fixed;
    bottom: 0;
    height: 40px;
}

#block-footer a:hover {
    color: steelblue;
    text-shadow: 0 1px 0 rgba(255, 255, 255, 0.2);
}
```
<br>
<p>Every set page had it's own color scheme to fit the theme of whatever set it was.<br>
For example set 2 had a pinkish red header color with black shadows. All the paragraph<br>
text would either be black or white with shadow effects based on the darkness or lightness<br>
of the background image.<br></p>
```
.set2h {
    color: #FA8072;
    text-shadow: 2px 2px black;
}
.set2pa {
    text-shadow: .5px .5px lightgray;
}
```
<br>
<h2>Other Skills Learned</h2>
<ul>
  <li>Learning a new API was an exciting experience. All API's are built differently,<br>
  so I had to be sure to read the documentation about the syntax and structure of the<br>
  API. It came out to be a fantastic learning experience, especially because the<br>
  specific API I chose happened to be strangely structured compared to others.<br>
  </li>
  <li>A great thing I learned during this project was to ask questions if I have<br>
  them. It's worse to sit there frustrated making no progress on a piece of code<br>
  than to simply reach out to someone and ask if they would be willing to help.<br>
  This taught me a lot about communication and being open about my progress.<br>
  I was also able to see what I was doing wrong and what I could do better.</li>
</ul>
