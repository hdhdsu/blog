Realm数据库入门
=====
##### 为什么用realm
android开发不仅仅是从网上获取数据并且展示,有时候我们需要把数据保存到我们本地,一般我们会想到用file,sp或者数据库sqlite,file的方式主要是IO流操作,sp是其实是使用xml的方式以键值对形式存储基本数据类型的数据。但是对于app中有复杂筛选查询的操作，file和sp都不能满足了.虽然sqlite可以满足有大量复杂查询要求的缓存数据操作。但是sqlite的使用略复杂，代码量很大.这时候就需要一种轻量级的数据库.Realm是一个可以替代SQLite以及ORM Libraries的轻量级数据库。

##### realm的优势
* 相比SQLite，Realm更快并且具有很多现代数据库的特性，比如除了基本数据类型之外,还支持JSON.流式api，数据变更通知，以及加密支持
* realm是一个跨平台移动数据库引擎，支持iOS、OS X（Objective-C和Swift）以及Android。这些都为移动端开发者带来了方便。
* 性能比较高
* 使用简单

##### 使用方法
1.在项目的build.grade中配置

	apply plugin: 'realm-android'
	buildscript {
	    repositories {
	        jcenter()
	        maven { url 'https://jitpack.io' }
	    }
	    dependencies {
	        classpath "io.realm:realm-gradle-plugin:1.0.0"
	    }
	}

2.初始化Realm,并进行相关配置.(Realm是框架的核心所在，是我们构建数据库的访问点,使用建造者模式构建对象。)

默认配置,

	public class MyApplication extends Application {
	  @Override
	  public void onCreate() {
	    super.onCreate();
	    // The Realm file will be located in Context.getFilesDir() with name "default.realm"
	    Realm.init(this);//初始化realm
	    RealmConfiguration config = new RealmConfiguration.Builder().build();//默认配置
	    Realm.setDefaultConfiguration(config);
	  }
	}

自定义配置

		public class MyApplication extends Application {
	  @Override
	  public void onCreate() {
	    super.onCreate();
	    Realm.init(this);//初始化
		//自定义配置
	    RealmConfiguration config = new  RealmConfiguration.Builder()
	                                         .name("myRealm.realm")
	                                         .deleteRealmIfMigrationNeeded()
	                                         .build();
	    Realm.setDefaultConfiguration(config);
	  }
	}

3.创建实体RealmObject(这是我们自定义的realm数据模型。创建数据模型的行为将会影响到数据库的结构。要创建一个数据模型，我们只需要继承RealmObject，然后设计我们想要存储的属性即可。)

	public class Dog extends RealmObject {
    private String name;
    private int age;
    
    @PrimaryKey
    private String id;


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }
	}

4.增

a.使用事务操作,新建一个对象,并且进行存储
	Realm realm=Realm.getDefaultInstance();
	
	realm.beginTransaction();//开启一个事务
	User user = realm.createObject(User.class); //创建一个对象
	user.setName("John");
	user.setEmail("john@corporation.com");
	realm.commitTransaction();//提交事务
	
b.使用事务块
	Realm  mRealm=Realm.getDefaultInstance();
	
	final User user = new User("John");//创建对象
	user.setEmail("john@corporation.com");
	
	mRealm.executeTransaction(new Realm.Transaction() {//事务块
	            @Override
	            public void execute(Realm realm) {
	            
	            realm.copyToRealm(user);
	               
	            }
	        });

5.删. (RealmResults：这个类是执行任何查询请求后所返回的类，其中包含了一系列的Object对象。和List类似，我们可以用下标语法来对其进行访问，并且还可以决定它们之间的关系。不仅如此，它还拥有许多更强大的功能，包括排序、查找等等操作。
)
 	
	Realm  mRealm=Realm.getDefaultInstance();

    final RealmResults<Dog> dogs=  mRealm.where(Dog.class).findAll();

        mRealm.executeTransaction(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
            
                Dog dog=dogs.get(5);
                dog.deleteFromRealm();
                //删除第一个数据
                dogs.deleteFirstFromRealm();
                //删除最后一个数据
                dogs.deleteLastFromRealm();
                //删除位置为1的数据
                dogs.deleteFromRealm(1);
                //删除所有数据
                dogs.deleteAllFromRealm();
            }
        });

上面的代码是用事务块进行删除,跟增加操作一样,也可以用事务beginTransaction和commitTransaction方法进行删除

6.改

RealmObject是自动更新的，比如你将一个数据读取出来后，又做了修改，修改会马上在数据中生效，不需要你再
将数据保存到数据库

	Realm  mRealm=Realm.getDefaultInstance();
	
	Dog dog = mRealm.where(Dog.class).equalTo("id", id).findFirst();
	mRealm.beginTransaction();
	dog.setName(newName);
	mRealm.commitTransaction();
也可以用事务块来进行修改

7.查

a.查询全部

	 public List<Dog> queryAllDog() {
	        Realm  mRealm=Realm.getDefaultInstance();
	    
	        RealmResults<Dog> dogs = mRealm.where(Dog.class).findAll();
	        
	        return mRealm.copyFromRealm(dogs);
	    }

b.条件查询

	  public Dog queryDogById(String id) {
        Realm  mRealm=Realm.getDefaultInstance();
    
        Dog dog = mRealm.where(Dog.class).equalTo("id", id).findFirst();
        return dog;
    }
还有其他的条件:

*	between(), greaterThan(), lessThan(), greaterThanOrEqualTo() & lessThanOrEqualTo()

*	equalTo() & notEqualTo()

*	contains(), beginsWith() & endsWith()

*	isNull() & isNotNull()

*	isEmpty() & isNotEmpty()

8.使用realm来操作数据库，尽管这些操作很快，但是还是建议放在后台进行。可以在回调中更新ui

	realm.executeTransactionAsync(new Realm.Transaction() {  
            @Override  
            public void execute(Realm bgRealm) {  
                User user = bgRealm.createObject(User.class);  
                user.setName("John");  
                user.setEmail("john@corporation.com");  
            }  
        }, new Realm.Transaction.OnSuccess() {  
            @Override  
            public void onSuccess() {  
                // Transaction was a success.  
            }  
        }, new Realm.Transaction.OnError() {  
            @Override  
            public void onError(Throwable error) {  
                // Transaction failed and was automatically canceled.  
            }  
        });

最后,我们要及时关闭realm任务，避免程序crash

	public void onDestroy () {  
    if (transaction != null && !transaction.isCancelled()) {  
        transaction.cancel();  
    }  
	}

##### Realm的一些坑

1.查询到的结果数据 不能跨线程使用.例如在主线程中查到的结果 在线程中想使用 需要在子线程中再查一次

2.realm对象不能跨线程使用.主线程中获取的realm对象, 不能在子线程中使用.需要重新调用getDefaultInstance()在子线程中获取另一个realm对象

3.当query到的数据要被使用时, realm数据库是要在开启的状态下的, 这样数据才能通过数据库链接访问到.在数据使用完毕后 需要把数据库关闭.一般选在在页面onDestory()的方法中 调用realm.close()方法关闭数据库链接.但如果网络请求数据的回调在页面关闭之后发生 也就是realm.close()调用了之后 又使用了同一个realm数据库链接查询数据
此时回报错误This Realm instance has already been closed, making it unusable.
解决办法是
如果还想使用同一个realm链接
先判断realm.isClosed()
数据库链接是否关闭了 如果关闭了重新打开一个新的数据库链接 使用完之后再将新的数据库链接关闭

4.如果想在Realm.close()之后继续操作查询得到的对象，只能复制一份数据传出来.

5.如果直接修改或删除query得到的数据，必须在transaction中完成.
