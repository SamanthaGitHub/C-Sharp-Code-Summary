## C# Live Project Code Summary
# Introduction
During my time studying at The Tech Academy, I was fortunate to complete a two week long Live Project. Much of the purpose of this project was to closely replciate a real-world working environment.

Using a Code First Entity Framework, I worked with my team members to develop a full scale MVC Web Application in C#. Our team used Azure DevOps and Agile methodology to plan and organize our project stories. I became much more familiar with the features of Visual Studio, as well as how to add appropriate functionality in a legacy codebase.

Featured below are some of my front end and back end code snippets that showcase the developmental process and functionality of this web application. My subset of tasks focused on rental capabilities for a theater company, creating a database of objects for rentable items (rooms and equipment).

[add info about styling about page; CSS; screenshot of gif?]

# MVC
## Model
In order to easily create the CRUD functions I needed, I started by building a model that I could then build the functionality and database from (Code First). Entity Framework made this task relatively straightforward. I had a parent class that two other classes inherited from.

```
public class Rental
    {
        [Display(Name = "ID")]
        public int RentalId { get; set; }
        [Display(Name = "Name")]
        public string RentalName { get; set; }
        [Display(Name = "Cost per hour")]
        public int RentalCost { get; set; }
        [Display(Name = "Current flaws or damages")]
        public string FlawsAndDamages { get; set; }

        public ERentalType RentalOptions { get; set; }
    }

    public class RentalEquipment : Rental
    {
        [Display(Name = "Choking hazard")]
        public bool ChokingHazard { get; set; }
        [Display(Name = "Sufocation hazard")]
        public bool SuffocationHazard { get; set; }
        [Display(Name = "Purchase price")]
        public int? PurchasePrice { get; set; }
    }

    public class RentalRoom : Rental
    {
        [Display(Name = "Room number")]
        public int? RoomNumber { get; set; }
        [Display(Name = "Square footage of room")]
        public int? SquareFootage { get; set; }
        [Display(Name = "Maximum occupancy of room")]
        public int? MaxOccupancy { get; set; }
    }

    public enum ERentalType
    {
        Rental,
        //[Display(Name = "Rental Equipment")]
        RentalEquipment,
        //[Display(Name = "Rental Room")]
        RentalRoom
    }
```

## Controller
After building my model and creating the initial CRUD functions provided by the Entity Framework, I then began to add logic so that those functions would operate appropriately. Much of the challenge for me came from finding a means to connect the model fields from my classes into one database entry, and having it register as either a rental, rental equipment, or a rental room. Below is how the post method of the create function was built.
### Create
I used if statements to check the type of rental and assign the appropriate variables.

```
public ActionResult Create([Bind(Include = "RentalId,RentalName,RentalCost,FlawsAndDamages,ChokingHazard,SuffocationHazard,PurchasePrice,RoomNumber,SquareFootage,MaxOccupancy")] Rental rental, RentalEquipment rentalEquipment, RentalRoom rentalRoom)

if (rentalEquipment.PurchasePrice != null)
{
    var rentalEquipments = new RentalEquipment
    {
        RentalName = rental.RentalName,
        RentalCost = rental.RentalCost,
        FlawsAndDamages = rental.FlawsAndDamages,
        ChokingHazard = rentalEquipment.ChokingHazard,
        SuffocationHazard = rentalEquipment.SuffocationHazard,
        PurchasePrice = rentalEquipment.PurchasePrice
    };

    db.Rentals.Add(rentalEquipments);
    db.SaveChanges();
    return RedirectToAction("Index");
}

if (rentalRoom.RoomNumber != null)
{
    var rentalRooms = new RentalRoom
    {
        RentalName = rental.RentalName,
        RentalCost = rental.RentalCost,
        FlawsAndDamages = rental.FlawsAndDamages,
        RoomNumber = rentalRoom.RoomNumber,
        SquareFootage = rentalRoom.SquareFootage,
        MaxOccupancy = rentalRoom.MaxOccupancy
    };

    db.Rentals.Add(rentalRooms);
    db.SaveChanges();
    return RedirectToAction("Index");
}

if (rentalRoom.RoomNumber == null && rentalEquipment.PurchasePrice == null)
{
    db.Rentals.Add(rental);
    db.SaveChanges();
    return RedirectToAction("Index");
}

return View(rental);
```
### Edit
The program's edit function was built slightly differently to better interact with the respective view.
```
public ActionResult Edit([Bind(Include = "RentalId,RentalName,RentalCost,FlawsAndDamages")] Rental rental, int? PurchasePrice, bool? ChokingHazard, bool? SuffocationHazard, int? RoomNumber, int? SquareFootage, int? MaxOccupancy)

if (PurchasePrice > 0)
{

    RentalEquipment rentalEquipment = new RentalEquipment();
    rentalEquipment.RentalId = rental.RentalId;

    rentalEquipment.RentalName = rental.RentalName;
    rentalEquipment.RentalCost = rental.RentalCost;
    rentalEquipment.FlawsAndDamages = rental.FlawsAndDamages;
    rentalEquipment.ChokingHazard = Convert.ToBoolean(ChokingHazard);
    rentalEquipment.SuffocationHazard = Convert.ToBoolean(SuffocationHazard);
    rentalEquipment.PurchasePrice = Convert.ToInt32(PurchasePrice);

    db.Entry(rentalEquipment).State = EntityState.Modified;
    db.SaveChanges();
    return RedirectToAction("Index");
}

if (RoomNumber > 0)
{
    RentalRoom rentalRoom = new RentalRoom();
    rentalRoom.RentalId = rental.RentalId;

    rentalRoom.RentalName = rental.RentalName;
    rentalRoom.RentalCost = rental.RentalCost;
    rentalRoom.FlawsAndDamages = rental.FlawsAndDamages;
    rentalRoom.RoomNumber = Convert.ToInt32(RoomNumber);
    rentalRoom.SquareFootage = Convert.ToInt32(SquareFootage);
    rentalRoom.MaxOccupancy = Convert.ToInt32(MaxOccupancy);

    db.Entry(rentalRoom).State = EntityState.Modified;
    db.SaveChanges();
    return RedirectToAction("Index");
}

if (ModelState.IsValid)
{
    db.Entry(rental).State = EntityState.Modified;
    db.SaveChanges();
    return RedirectToAction("Index");
}
return View(rental);

```
## View

A common layout page was implemented in creating the views, keeping the site in a uniform style while navigating the pages. The layout page was a component of the legacy codebase, ensuring cohesive development across all components. I created javascript functions and used an enum to hide and show the available input fields, depending on the type of object being created.
### Create
#### Dropdown Selection
Depending on the enum value chosen, the compatible fields will display:

```
@Html.DropDownListFor(model => model.RentalOptions,
        new SelectList(Enum.GetValues(typeof(ERentalType))),
        "Select rental type:"
        )
```
### Edit
In order to show compatible fields in the edit view, the program decifers the model type:
```
@if (Model.GetType() == typeof(RentalEquipment))
{...}
else if (Model.GetType() == typeof(RentalRoom))
{...}
```


## Skills Aquired
### <b>MVC</b>
This project was one my first larger MVC applications. Being able to understand how this framework is built and connecting 'under the hood' is vital to a functioning finished product. The MVC framework aided me in many ways, one of which being the seperation of conerns. When adding in features or designing pages, I was able to focus on a few specific areas of my code, allowing the framework to do the rest.

### <b>Version Control</b>
Git Version control was used extensively, as all team members worked off of cloned local versions of the same remote repository. I created branches for each of my stories and pushed them to the project manager. When merge conflicts arose, we were able to see exactly where the conflicts were and address them. I also found Git to be a great tool for debugging, as I was able to commit my branch for my project manager to then see and help me figure out problems.

### <b>Troubleshooting</b>
When running into roadblocks, I frequently searched online for answers. By using various search terms and looking at different search results, I became more adept at finding exactly what I was after. I also learned how useful looking at a variety of answers to a problem is, as I could cherry-pick different aspects I wanted to implement into my code. Using Chrome Developer Tools allowed me to see exactly what my application was doing in the browser, which files were being used, and even some possible errors. Communicating with my project manager was also a great resource I had available, and was able to see how different ideas came together to reach a common goal.
