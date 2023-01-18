## 關聯映射
>在關係型數據庫中，多表之間存在著三種關聯關係，分別為一對一、一對多和多對多
>
>關聯關係描述的是數據庫表之間的引用關係，而Hibernate 關聯映射指的就是與數據庫表對應的實體類之間的引用關係。
>
>在數據庫中，如果兩張表想要建立關聯關係，就需要外鍵來連接它們，數據庫表之間的關係是沒有方向性的，且彼此是透明的。而在Java 中，如兩個類想要建立關係的話，那就需要這兩個類都通過屬性（變量）來管理對方的引用，以達到建立關聯關係的目的，Hibernate 關聯映射也是通過這種方式實現的。

**創建InstructorDetail實體類別(`副表`)**
```
@Entity
@Table(name="instructor_detail")
public class InstructorDetail {
	
	// annotate the class as an entity and map to db table
	
	// define the fields
	
	// annotate the fields with db column names
	
	// create constructors
	
	// generate getter/setter methods
	
	// generate toString() method
	
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="id")
	private int id;
	
	@Column(name="youtube_channel")
	private String yputubeChannel;
	
	@Column(name="hobby")
	private String hobby;
	
	public InstructorDetail()
	{
		
	}

	public InstructorDetail(String yputubeChannel, String hobby) {
		this.yputubeChannel = yputubeChannel;
		this.hobby = hobby;
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getYputubeChannel() {
		return yputubeChannel;
	}

	public void setYputubeChannel(String yputubeChannel) {
		this.yputubeChannel = yputubeChannel;
	}

	public String getHobby() {
		return hobby;
	}

	public void setHobby(String hobby) {
		this.hobby = hobby;
	}

	@Override
	public String toString() {
		return "InstructorDetail [id=" + id + ", yputubeChannel=" + yputubeChannel + ", hobby=" + hobby + "]";
	}
}
```
**創建Instructor實體類別(`主表`)**
>參考網址：https://juejin.cn/post/6963550783132893198
>
>Cascade 級聯關係：實際業務中，我們通常會遇到以下情況：用戶和用戶的收貨地址是一對多關係，當用戶被刪除時，這個用戶的所有收貨地址也應該一併刪除。訂單和訂單中的商品也是一對多關係，但訂單被刪除時，訂單所關聯的商品肯定不能被刪除。此時只要配置正確的級聯關係，就能達到想要的效果。級聯關係類型：
>
>* CascadeType.REFRESH：級聯刷新，當多個用戶同時作操作一個實體，為了用戶取到的數據是實時的，在用實體中的數據之前就可以調用一下refresh()方法
>* CascadeType.REMOVE：級聯刪除，當調用remove()方法刪除Order實體時會先級聯刪除OrderItem的相關數據
>* CascadeType.MERGE：級聯更新，當調用了Merge()方法，如果Order中的數據改變了會相應的更新OrderItem中的數據
>* CascadeType.ALL：包含以上所有級聯屬性
>* CascadeType.PERSIST：級聯保存，當調用了Persist() 方法，會級聯保存相應的數據
>
>@OneToOne 用來表示類似於Instructor與InstructorDetail之間的一對一的關係，在Instructor表中會有一個指向InstructorDetail表主鍵的字段instructor_detail_id，所以主控方（指能夠主動改變關聯關係的一方）一定是Instructor，因為，只要改變Instructor表的instructor_detail_id就改變了Instructor與InstructorDetail之間的關聯關係，所以@JoinColumn要寫在Instructor實體類上，自然而然地，InstructorDetail就是被控方了
```
@Entity
@Table(name="instructor")
public class Instructor {

	// annotate the class as an entity and map to db table
	
	// define the fields
		
	// annotate the fields with db column names
	
	// ** set up mapping to InstructorDetail entity
		
	// create constructors
		
	// generate getter/setter methods
		
	// generate toString() method	
	
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="id")
	private int id;
	
	@Column(name="first_name")
	private String firstName;
	
	@Column(name="last_name")
	private String lastName;
	
	@Column(name="email")
	private String email;
	
	@OneToOne(cascade=CascadeType.ALL)
	@JoinColumn(name="instructor_detail_id")
	private InstructorDetail instructorDetail;
	
	public Instructor()
	{
		
	}

	public Instructor(String firstName, String lastName, String email) {
		this.firstName = firstName;
		this.lastName = lastName;
		this.email = email;
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

	public InstructorDetail getInstructorDetail() {
		return instructorDetail;
	}

	public void setInstructorDetail(InstructorDetail instructorDetail) {
		this.instructorDetail = instructorDetail;
	}

	@Override
	public String toString() {
		return "Instructor [id=" + id + ", firstName=" + firstName + ", lastName=" + lastName + ", email=" + email
				+ ", instructorDetail=" + instructorDetail + "]";
	}
}
```
**演示創建導師與導師詳細資料以及級聯同步動作**
```
public class CreateDemo {

	public static void main(String[] args) {
		
		// create session factory
		SessionFactory factory = new Configuration()
				.configure("hibernate.cfg.xml")
				.addAnnotatedClass(Instructor.class)
				.addAnnotatedClass(InstructorDetail.class)
				.buildSessionFactory();
		
		// create session
		Session session = factory.getCurrentSession();
		
		try {  
			Instructor tempInstructor = 
					new Instructor("Madhu", "Patel", "madhu@luv2code.com");
			
			InstructorDetail tempInstructorDetail = 
					new InstructorDetail(
							"http://www.youtube.com",
							"Guitar");
			
			// associate the objects
			tempInstructor.setInstructorDetail(tempInstructorDetail);
			
            // start transaction
            session.beginTransaction();
            
            // save the instructor
            // Note: this will ALSO save the details object
            // because of CascadeType.ALL
            //
            session.save(tempInstructor);
            
            // commit transaction
            session.getTransaction().commit();
        } catch (Exception exc) {
            exc.printStackTrace();
        } finally {
            factory.close();
        }
	}
}
```
**演示刪除導師與導師詳細資料以及級聯同步動作**
```
public class DeleteDemo {

	public static void main(String[] args) {
		
		// create session factory
		SessionFactory factory = new Configuration()
				.configure("hibernate.cfg.xml")
				.addAnnotatedClass(Instructor.class)
				.addAnnotatedClass(InstructorDetail.class)
				.buildSessionFactory();
		
		// create session
		Session session = factory.getCurrentSession();
		
		try {  				
            // start transaction
            session.beginTransaction();
            
            // get instructor by primary key / id
            int theId = 1;
            
            // Will return null If not found
            Instructor tempInstructor = 
            		session.get(Instructor.class, theId);
            
            // delete the instructors
            if (tempInstructor != null) {
				// Note: will ALSO delete associated "details" object
				// because of CascadeType.ALL 
				session.delete(tempInstructor);
			}

            // commit transaction
            session.getTransaction().commit();
            
        } catch (Exception exc) {
            exc.printStackTrace();
        } finally {
            factory.close();
        }
	}
}
```
