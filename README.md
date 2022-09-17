# SET-UP 

    cd Desktop
    rails new wildlife-api -d postgresql -T
    cd wildlife-api
    rails db:create

#### Add the remote from GitHub

    git remote add origin https://github.com/learn-academy-2022-echo/wildlife-tracker-CathrineC.git

#### Create the main branch

    git add .

#### Make an initial commit

    git commit -m "initial commit"

    rails s
    git push origin main
    git branch
    git checkout -b animal-crud-actions
    bundle add rspec-rails
    rails generate rspec:install
    code .

### Wildlife Tracker Challenge

##### The Forest Service is considering a proposal to place in conservancy a forest of virgin Douglas fir just outside of Portland, Oregon. Before they give the go ahead, they need to do an environmental impact study. They've asked you to build an API the rangers can use to report wildlife sightings.


### Story 1: In order to track wildlife sightings, as a user of the API, I need to manage animals.

#### Branch: animal-crud-actions >> terminal >>

    it checkout -b animal-crud-actions

## Acceptance Criteria

### Create a resource for animal with the following information: common name and scientific binomial

#### terminal >>

    rails g resource Animal 
    common_name:string scientific_binomial:string

#### to see all available routes >> terminal >>
    
    rails routes

#### migrate database (shows in schema.rb) >> terminal >>

    rails db:migrate

#### add item in the database >> terminal >>
    rails c

    Animal.create common_name:'Animal1', scientific_binomial:'animal1 binomial'

### Can see the data response of all the animals
#### POSTMAN: animals GET /animals(.:format) animals#index
#### class AnimalsController < ApplicationController

    def index
    animals = Animal.all
    render json: animals
    end

    def show
    animal = Animal.find(params[:id])
    render json: animal
    end

### Can create a new animal in the database
#### class AnimalsController < ApplicationController

    def create
        animal = Animal.create(animal_params)
        if animal.valid?
            render json: animal
        else
            render json: animal.errors
        end
    end
   
   private
        def animal_params
            params.require(:animal).permit(:common_name,:scientific_binomial)
        end

#### POSTMAN: POST   /animals(.:format) animals#create
#### POSTMAN: body >> raw >> JSON

    {
    "animal": {
        "common_name":"Animal3", 
        "scientific_binomial":"animal3 binomial"
    }
    }

#### 422 Unprocessable Entity >> add in ApplicationController < ActionController::Base

    class ApplicationController < ActionController::Base
    skip_before_action :verify_authenticity_token
    end

### Can update an existing animal in the database
#### class AnimalsController < ApplicationController

    def create
        animal = Animal.create(animal_params)
        if animal.valid?
            render json: animal
        else
            render json: animal.errors
        end
    end
   
    private
    def animal_params
        params.require(:animal).permit(:common_name,:scientific_binomial)
    end

#### POSTMAN: PATCH  /animals/:id(.:format)animals#update

### Can remove an animal entry in the database

#### class AnimalsController < ApplicationController

   def destroy
        animal = Animal.find(params[:id])
        if animal.destroy
            render json: animal
        else
            render json: animal.errors
        end
   end

#### POSTMAN: DELETE /animals/:id(.:format)animals#destroy


### Story 2: In order to track wildlife sightings, as a user of the API, I need to manage animal sightings.

#### Branch: sighting-crud-actions >> terminal >>

    git checkout -b sighting-crud-actions

### Acceptance Criteria

### Create a resource for animal sightings with the following information: latitude, longitude, date. Hint: An animal has_many sightings (rails g resource Sighting animal_id:integer ...).Hint: Date is written in Active Record as yyyy-mm-dd (“2022-07-28")

#### >> terminal >>

    rails g resource Sighting animal_id:integer latitude:float longitude:float date
    :date

#### migrate database (shows in schema.rb) >> terminal >>

    rails db:migrate

#### to see all available routes >> terminal >>
    
    rails routes

#### add item in the database >> terminal >>

    rails c

    Sighting.create animal_id:1, latitude:38.8951, longitude:-77.0364, date:211231 

#### date data type format: YYYYMMDD

### Can create a new animal sighting in the database

#### SightingsController < ApplicationController

    def create
        sighting = Sighting.create(sighting_params)
        if sighting.valid?
            render json: sighting
        else
            render json: sighting.errors
        end
    end

    private
    def sighting_params
        params.require(:sighting).permit(:animal_id, :latitude, :longitude, :date)
    end

#### POSTMAN: POST   /sightings(.:format)sightings#create

### Can update an existing animal sighting in the database

#### SightingsController < ApplicationController

    def update
        sighting = Sighting.find(params[:id])
        sighting.update(sighting_params)
        if sighting.valid?
            render json: sighting
        else
            render json: sighting.errors
        end
    end

    private
    def sighting_params
        params.require(:sighting).permit(:animal_id, :latitude, :longitude, :date)
    end

####  POSTMAN: PATCH  /sightings/:id(.:format)sightings#update

### Can remove an animal sighting in the database

#### SightingsController < ApplicationController

    def destroy
        sighting = Sighting.find(params[:id])
        if sighting.destroy
            render json: sighting
        else
            render json: sighting.errors
        end
    end

#### POSTMAN: DELETE /sightings/:id(.:format)sightings#destroy

### Story 3: In order to see the wildlife sightings, as a user of the API, I need to run reports on animal sightings.

#### Branch: animal-sightings-reports >> terminal >>

### Acceptance Criteria

#### Can see one animal with all its associated sightings
#### Hint: Checkout this example on how to include associated records

##### AnimalsController < ApplicationController

    def show
        animal = Animal.find(params[:id])
        render json: animal, include:[:sightings]
    end

##### SightingsController < ApplicationController

    def show
        sighting = Sighting.find_by(id: params[:id])
        if sighting
            render json: sighting.to_json(include: [:animals])
        else
            render json: { message: 'There is no sighting for the selected animal.' }
        end        
    end

##### sighting.rb
    class Sighting < ApplicationRecord
        belongs_to:animal
    end
##### animal.rb
    class Animal < ApplicationRecord
        has_many:sightings
    end

#### Can see all the all sightings during a given time period
##### Hint: Your controller can use a range to look like this:
    class SightingsController < ApplicationController
    def index
        sightings = Sighting.where(date: params[:start_date]..params[:end_date])
        render json: sightings
    end
    end

##### Hint: Be sure to add the start_date and end_date to what is permitted in your strong parameters method

    def sighting_params
        params.require(:sighting).permit(:animal_id, :latitude, :longitude, :start_date, :end_date)
    end

##### Hint: Utilize the params section in Postman to ease the developer experience
##### Hint: Routes with params

    Rails.application.routes.draw do
        resources :sightings
        resources :animals
    end


### Stretch Challenges

### Story 4: In order to see the wildlife sightings contain valid data, as a user of the API, I need to include proper specs.

#### Branch: animal-sightings-specs

### Acceptance Criteria
Validations will require specs in spec/models and the controller methods will require specs in spec/requests.

Can see validation errors if an animal doesn't include a common name and scientific binomial

Can see validation errors if a sighting doesn't include latitude, longitude, or a date

Can see a validation error if an animal's common name exactly matches the scientific binomial

Can see a validation error if the animal's common name and scientific binomial are not unique

Can see a status code of 422 when a post request can not be completed because of validation errors

Hint: Handling Errors in an API Application the Rails Way

### Story 5: In order to increase efficiency, as a user of the API, I need to add an animal and a sighting at the same time.

#### Branch: submit-animal-with-sightings

### Acceptance Criteria

#### Can create new animal along with sighting data in a single API request
#### Hint: Look into accepts_nested_attributes_for