
# Python Live Project
During my sprint, I worked in a team to complete stories on a Python Live Project using the Django Framework. This was a full-stack project to create a hobby tracking app. I worked on an app to keep track of pets, specifically cats. This can easily be altered to include different types of pets. User input is obtained to add the pets to the database, as shown below. I implemented CRUD functionality. I utilised function-based views in this app.
## Tech used
For this project, I used Django, Python, HTML and CSS. We used Azure Devops in conjunction with Git for version control. The IDE used was Pycharm.

## Features
### Create
I first needed to determine what data a user may want to store about their pet. I created a [class](https://github.com/dhavap/Python-Live-Project/blob/master/models.py) that contained fields for data I wanted to store such as age, medical conditions, and vet details. I used the modelform class to quickly create a [form](https://github.com/dhavap/Python-Live-Project/blob/master/forms.py) for users to fill in.
```
def add_cat(request): # add cat to db
    form = CatForm(request.POST or None)
    if form.is_valid():
        form.save()
        return redirect('catList')
    else:
        print(form.errors)
        form = CatForm
    return render(request, 'CatCare/cat_create.html', {'form': form})
```

A limited number of details of the cats saved to the database are displayed on the index page. A link in each row displays the complete information about each cat.

### Read
I created 2 different functions to allow the user to view stored data. The first, named the index page in this app, queried the database for all items stored in the table , which was then rendered on a [template](https://github.com/dhavap/Python-Live-Project/blob/master/cat_index.html).
```
def index(request):  # list of cats in db
    get_cats = Cat.Cats.all()
    print(Cat.cat_name)
    context = {'cats': get_cats}
    return render(request, 'CatCare/cat_index.html', context)
```
My next task was to create a details page, should a user choose to view further details about a single pet. Here, I needed to pass a variable, in this case the pet's id, through the url from the index page. I queried the database to obtain data about the cat chosen by the user.
```
def details_cat(request, id): # display cat details onclick from index page
    id = int(id)
    cat = get_object_or_404(Cat, pk=id)
    context = {'cat': cat}
    return render(request, 'CatCare/cat_details.html', context)
```
### Update
Users also needed to be able to update information about their pets, such as if the pet's vet has changed. 
```
def edit_cat(request, id): # edit cat details
    id = int(id)
    cat = get_object_or_404(Cat, pk=id)
    form = CatForm(request.POST or None, instance=cat)
    if form.is_valid():
        form.save()
        return redirect('catList')
    else:
        print(form.errors)
    context = {'form':form}
    return render(request, 'CatCare/cat_edit.html', context)
```

### Delete
Lastly, the user may also wish to stop tracking the cat. In which case, they may wish to delete the cat from the database. 
```
def delete_cat(request, id): # delete cat details
    id = int(id)
    cat = get_object_or_404(Cat, pk=id)
    if request.method == 'POST':
        cat.delete()
        return redirect('catList')
    context = {"cat":cat}
    return render(request, 'CatCare/cat_delete.html', context)
```

## Future possibilities
In future, I hope to expand upon this app to create a full project complete with user authentication. My intention is to eventually create a project that resembles the systems vet clinics use to keep track of their patients. 



# Back End Live Project
<h2>Introduction</h2>
As a student of The Tech Academy, I worked on Python live project for a 2-week sprint. I got to experience working on an ongoing project in a team of fellow software developers. This repository provides an overview of the stories I worked on. I used the Django framework for this project. 

I have included descriptions of the stories I worked on below, along with screenshots of my code snippets from the project.I have also included some files containing the full code I created for this project in this repository.

## Tech used
For this story, I used Python, Django, and Beautiful Soup. Azure Devops was used to manage the project. The IDE used was Visual Studio Code.

## Features
### Web Scraping
I was tasked with extracting data from a Wikipedia page on the timeline of space exploration. I created a dropdown list allowing users to select the decade of space exploration they wished to view. Upon receiving user input, I then used Beautiful Soup to parse and extract the relevant data from the webpage. This was a new technology I had to learn to complete this story. I then used Pandas to create a table out of the parsed data and rendered this table display on the result page. 


```
#============== Rendering page with decade options for user to choose from
def space_exploration(request):
    decades = {'decades' : DECADE_OPTIONS}
    return render(request, 'explorationTimeline/space_exploration.html', decades)

#============== Using BeautifulSoup to get tables from Wikipage based on user selected decade
def result(request, decade):  #pass the user selected variable 'decade' into the function
    print("Decade selected: " + str(decade)) #prints to console for development purposes
    getPage = requests.get("https://en.wikipedia.org/wiki/Timeline_of_Solar_System_exploration") #get the page
    src = getPage.content #store the page's contents as a variable
    soup = BeautifulSoup(src, 'html.parser') #parse the content of the page
    missionsTable = soup.find(id= decade).findNext('table', {"class" : "wikitable"}) # get the requested table by looking at the table after the table headline   
    #print(missionsTable)

    create_table = [['Mission Name', 'Launch Date', 'Country', 'Wikipage']] #create list to put extracted data into
    rows = missionsTable.find_all('tr')[1:] #extract all rows except the first row because it is a header

    for row in rows: #loop through all the rows to extract desired data
        if decade in ('1950s', '1960s', '1970s'):
            mission_name = row.find_all('a')[1].get_text() #extract mission name
            wiki_link = 'https://en.m.wikipedia.org' + row.find_all('a')[1].get('href') # extract href
        elif decade in ('1980s', '1990s', '2000s', '2010s', 'Planned_or_scheduled'):
            mission_name = row.td.find('a', recursive= False).get_text() #extract mission name
            wiki_link = 'https://en.m.wikipedia.org' + row.td.find('a', recursive= False).get('href') # extract href
        launch_date = row.find_all('td')[1].get_text()[:-1]
        country = row.find('a').get('title')
        wikipage= '<a href="' + wiki_link + '">More Info</a>' # concatenate href to create link to mission wikipage
        display_row = [mission_name, launch_date, country, wikipage] # create a row
        create_table.append(display_row) # add the rows to the table

    df = pd.DataFrame(create_table[1:], columns = create_table[0]) #create table via Pandas module
    data = df.to_html(escape = False) #convert table to html
    #print(data)
    context = {
        'data' : data,
        'decade': decade,
    }
    return render(request, 'explorationTimeline/result.html', context)
```
