# bloc-steps-from-repo-firebase


## set up flutter for firebase
### 1 - create a method to convert data snap shot received from firebase to instance of the model class
### 2- create a repo class for each collection inside firebase 
### 3- create instance of `FirebaseFirestore` class 
### 4- initialize instance of `FirebaseFirestore` variable in the constructor 
### 5- use instance of `FirebaseFirestore` variable to retrieve collection and map it is data using `fromSnapshot` method to model class 

## create states
###### 1 - create abstract class 
###### 2 - extend states from this abstract class
###### 3 - each inherited state can recieve a variable value form .... remote or local repo 
## create events
###### 1 - create abstract class 
###### 2 - extend events from this abstract class
###### 3 - each inherited event can recieve a variable value form .... widget triggering event 
## connect event to state
###### 1 - create class extending from bloc class 
###### 2 - pass initial state object to the parent class 
###### 3 - in the body of the child constructor call ```on``` method which registers one event handler for each event 
###### 4 - create an event handler method for each event which takes different event classes and abstract state class 
###### 5 - depending on the event handler type which can be used for 
  - loading data 
    - use a sync and await 
    - emit a state for loaing data 
    - after one second emit a state that data is loaded 
    - catch errors in a state 
  - adding data 
    - store current state inside a variable 
    - check if state is equal to a specific state then you can emit a new state with the modification required which is added data from an event 
  - removing data
    - store current state inside a variable 
    - check if state is equal to a specific state then you can emit a new state with the modification required which is removed data from an event 
## create an object from the bloc and pass it an initial event object 
###### 1 - wrap material app with multi bloc provider 
###### 2 - use bloc provider to create the bloc object so that a single instance of a bloc can be provided to multiple widgets within a subtree.
###### 3 - pass it an initial event object 
## update UI based on state 
###### 1 - wrap widget updating state on the ui with a bloc builder widget 
###### 2 - data type will be object from the bloc and state 
###### 3 - check if the state is still loading so we can show circular progress indicator 
###### 3 - check if the state is loaded so we can show the actual data  
###### 4 - modify the data so so it reflects the data coming from state   
## take data input from ui as an event 
###### 1 - wrap widget triggering event on the ui with a bloc builder widget 
###### 2 - data type will be object from the bloc and state 
###### 3 - when the user press a button use context to read the bloc 
###### 4 - add event to it and pass the event the data it needs (ex: object)

*********************************************************************************************

# bloc-steps in details 

## set up flutter for firebase
### 1 - create a method to convert data snap shot received from firebase to instance of the model class
```dart
  static Category fromSnapshot(DocumentSnapshot snapshot) {
    Category category =
        Category(name: snapshot['name'], imageUrl: snapshot['imageUrl']);
    return category;
  }
}
```
### 2- create a repo class for each collection inside firebase 
```dart
abstract class BaseCategoryRepository {
  Stream<List<Category>> getAllCategories();
}
```
```dart
class CategoryRepository extends BaseCategoryRepository {
  @override
  Stream<List<Category>> getAllCategories() {
    //todo
    });
  }
}
```
### 3- create instance of `FirebaseFirestore` class 
```dart
class CategoryRepository extends BaseCategoryRepository {
  final FirebaseFirestore _firebaseFirestore;
  @override
  Stream<List<Category>> getAllCategories() {
    //todo
    });
  }
}
```
### 4- initialize instance of `FirebaseFirestore` variable in the constructor 
```dart
class CategoryRepository extends BaseCategoryRepository {
  final FirebaseFirestore _firebaseFirestore;
  // ? this can be null and if it is null we use  FirebaseFirestoe to create instance 
  CategoryRepository({FirebaseFirestore? firebaseFirestore})
      : _firebaseFirestore = firebaseFirestore ?? FirebaseFirestore.instance;
  @override
  Stream<List<Category>> getAllCategories() {
    //todo
    });
  }
}
```
### 5- use instance of `FirebaseFirestore` variable to retrieve collection and map it is data using `fromSnapshot` method to model class 
```dart
class CategoryRepository extends BaseCategoryRepository {
  final FirebaseFirestore _firebaseFirestore;
  // ? this can be null and if it is null we use  FirebaseFirestoe to create instance 
  CategoryRepository({FirebaseFirestore? firebaseFirestore})
      : _firebaseFirestore = firebaseFirestore ?? FirebaseFirestore.instance;
  @override
  Stream<List<Category>> getAllCategories() {
    return _firebaseFirestore
        .collection('categories')
        .snapshots()
        .map((snapshot) {
      return snapshot.docs.map((doc) => Category.fromSnapshot(doc)).toList();
    });
  }
}
```
## create states
### 1 - create abstract class 
```dart
abstract class CategoryState extends Equatable {
  const CategoryState();

  @override
  List<Object> get props => [];
}
```
### 2 - extend states from this abstract class
```dart
class CategoryLoading extends CategoryState {}

class CategoryLoaded extends CategoryState {
  @override
  List<Object> get props => [];
}
```
### 3 - each inherited state can recieve a variable value form .... corresponding event UpdateModel event `UpdateCategory`
```dart
class CategoryLoading extends CategoryState {}

class CategoryLoaded extends CategoryState {
  final List<Category> categories;
  const CategoryLoaded({this.categories = const <Category>[]});

  @override
  List<Object> get props => [categories];
}
```
## create events
### 1 - create abstract class 
```dart
abstract class CategoryEvent extends Equatable {
  const CategoryEvent();

  @override
  List<Object> get props => [];
}
}
```
### 2 - extend events from this abstract class
```dart
class LoadCategories extends CategoryEvent {}

class UpdateCategory extends CategoryEvent {
  List<Object> get props => [];
}

```
### 3 - update event will receive the data from the load event 
```dart
class LoadCategories extends CategoryEvent {}

class UpdateCategory extends CategoryEvent {
  final List<Category> categories;
  UpdateCategory(this.categories);
  List<Object> get props => [categories];
}

```
## connect event to state
### 1 - create class extending from bloc class 
```dart
class CategoryBloc extends Bloc<CategoryEvent, CategoryState> {
  CategoryBloc() : super() {
  }
```
### 2 - pass initial state object to the parent class 
```dart
class CategoryBloc extends Bloc<CategoryEvent, CategoryState> {
  CategoryBloc() : super(CategoryLoading()) {
  }
```
### 3 - in the body of the child constructor call ```on``` method which registers one event handler for each event 
```dart
class CategoryBloc extends Bloc<CategoryEvent, CategoryState> {
  CategoryBloc() : super(CategoryLoading()) {
    on<LoadCategories>(_onLoadCategories);
    on<UpdateCategory>(_onUpdateCategories);
  }
```
### 3 - recieve instance of the model repo class from the `main.dart` file 
```dart
class CategoryBloc extends Bloc<CategoryEvent, CategoryState> {
  final CategoryRepository _categoryRepository;
  StreamSubscription? _categorySubscription;
  CategoryBloc(CategoryRepository categoryRepository)
      : _categoryRepository = categoryRepository,
        super(CategoryLoading()) {
    on<LoadCategories>(_onLoadCategories);
    on<UpdateCategory>(_onUpdateCategories);
  }
```
### 5 - create an event handler method for each event which takes different event classes and abstract state class 
### 6 - depending on the event handler type which can be used for 
  - loading data from firestore 
    - use a sync 
    - cancel stream subscription of the model 
    - use instance of the model repo class to access it is base method which gets the data `getAllProducts()`
    - listen for the list received from the method and pass it to the next event `UpdateCategories`
    ```dart
  void _onLoadCategories(
      LoadCategories event, Emitter<CategoryState> emit) async {
    _categorySubscription?.cancel();
    _categorySubscription = _categoryRepository.getAllCategories().listen(
          (categories) => add(
            UpdateCategory(categories),
          ),
        );
  }
     ```
  - updating data from load data event 
    - use data from the load cateogries event 
    ```dart
  void _onUpdateCategories(UpdateCategory event, Emitter<CategoryState> emit) {
    emit(CategoryLoaded(categories: event.categories));
  }
   
## create an object from the bloc and pass it an initial event object 
### 1 - wrap material app with multi bloc provider 
```dart
return MultiBlocProvider(
      providers: [
        
      ],
      child: GetMaterialApp(
        debugShowCheckedModeBanner: false,
        title: 'Flutter',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: HomeScreen(),
      ),
    );
```
### 2 - use bloc provider to create the bloc object so that a single instance of a bloc can be provided to multiple widgets within a subtree.
### 3 - pass it an initial event object 
```dart
return MultiBlocProvider(
      providers: [
        BlocProvider(create: (_) => WishlistBloc()..add(LoadWishlist()))
      ],
      child: GetMaterialApp(
        debugShowCheckedModeBanner: false,
        title: 'Flutter second commit',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: HomeScreen(),
      ),
    );
```
## update UI based on state 
### 1 - wrap widget updating state on the ui with a bloc builder widget 
### 2 - data type will be object from the bloc and state 
### 3 - check if the state is still loading so we can show circular progress indicator 
### 3 - check if the state is loaded so we can show the actual data  
### 4 - modify the data so so it reflects the data coming from state   
```dart
body: BlocBuilder<WishlistBloc, WishlistState>(
        builder: (context, state) {
          if (state is WishlistLoading) {
            return Center(
              child: CircularProgressIndicator(),
            );
          }
          if (state is WishlistLoaded) {
            return GridView.builder(
                padding: EdgeInsets.symmetric(horizontal: 8.0, vertical: 16.0),
                gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                    crossAxisCount: 1, childAspectRatio: 1.7),
                itemCount: state.wishlist.products.length,
                itemBuilder: (BuildContext context, int index) {
                  return Center(
                    child: ProductCard(
                      product: state.wishlist.products[index],
                      widthFactor: 1.1,
                      isWishlist: true,
                    ),
                  );
                });
          } else {
            return Text('something went wrong');
```
## take data input from ui as an event 
### 1 - wrap widget triggering event on the ui with a bloc builder widget 
### 2 - data type will be object from the bloc and state 
### 3 - when the user press a button use context to read the bloc 
### 4 - add event to it and pass the event the data it needs (ex: object)
```dart
BlocBuilder<WishlistBloc, WishlistState>(
                builder: (context, state) {
                  return IconButton(
                    onPressed: () {
                      context
                          .read<WishlistBloc>()
                          .add(AddProductToWishlist(product));
                    },
                    icon: Icon(Icons.favorite),
                    color: Colors.black,
                  );
                },
              ),
```

