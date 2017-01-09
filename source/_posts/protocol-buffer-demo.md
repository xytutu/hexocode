---
title: Protocol Buffer Java 实例
date: 2017-01-05 11:04:09
tags: Protocol Buffer
---
Protocol Buffer 是Google一款序列化的方案。序列化是网络传输的基础，在现阶段分布式使用愈加广泛的背景下，高效的序列化方案是必须的。比如Rpc，Netty都有基于protocol buffer的实现。
今天写个简单的实例，看一下开发的全过程。

### 编写proto文件
  proto文件就是序列化的模板结构，后续proto编译器能够根据用户编写的模板文件生成相应的java类。
  proto文件是静态的，只需要描述文件结构就可以相当于贫血模式的代码；后续生成的java文件会自动生成，get/set/build/parse等Protocol Buffer解析的方法，相当于充血模式，通过这种方式用户就不必理解Protocol Buffer的解析规范去实现相应的方法，主需关注自己要序列化的结构。
  ```
  package bs.maven.test.protobuf;
  option java_package = "bs.maven.test.protobuf";
  option java_outer_classname = "AddressBookProtos";
  
  message Person {
    required string name = 1;
    required int32 id = 2;
    optional string email = 3;
  
    enum PhoneType {
      MOBILE = 0;
      HOME = 1;
      WORK = 2;
    }
  
    message PhoneNumber {
      required string number = 1;
      optional PhoneType type = 2 [default = HOME];
    }
  
    repeated PhoneNumber phone = 4;
  }
  
  message AddressBook {
    repeated Person person = 1;
  }
  ```
### 编译proto文件
```
  protoc -I=$SRC_DIR --java_out=$DST_DIR $SRC_DIR/addressbook.proto
```
[Mac上protoc编译器的安装参看](http://my.oschina.net/KingPan/blog/283881?fromerr=8vajR5S9)

### 使用proto文件

  将生成的AddressBookProtos代码直接拷贝到相应的目录就可以直接开发。以文件的方式表示序列化过程，网络传输同理。
  * 工程Maven 依赖：
  ```
  <dependency>
			<groupId>com.google.protobuf</groupId>
			<artifactId>protobuf-java</artifactId>
			<version>2.5.0</version>
		</dependency>
  ```
  * 实例代码
  ```
  public class AddressTest {

	private static final String fileName = "/Users/bianshuai/Documents/tmp/protobuf/address";

	public static void main(String[] args) {
		write();
		read();
	}

	public static void write() {

		AddressBookProtos.Person john = Person.newBuilder().setId(1234).setName("John Doe").setEmail("jdoe@example.com")
				.addPhone(Person.PhoneNumber.newBuilder().setNumber("555-4321").setType(Person.PhoneType.HOME)).build();

		AddressBookProtos.Person gavin = Person.newBuilder().setId(12345).setName("gavin Doe")
				.setEmail("gavin@example.com")
				.addPhone(Person.PhoneNumber.newBuilder().setNumber("555-4323").setType(Person.PhoneType.HOME)).build();

		AddressBookProtos.AddressBook address = AddressBookProtos.AddressBook.newBuilder().addPerson(john)
				.addPerson(gavin).build();
		FileOutputStream output = null;
		try {
			output = new FileOutputStream("fileName");
			address.writeTo(output);
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			if (null != output) {
				try {
					output.close();
				} catch (IOException e) {
				}
			}
		}
	}

	public static void read() {
		AddressBook addressBook;
		try {
			addressBook = AddressBook.parseFrom(new FileInputStream(fileName));
			Print(addressBook);
		} catch (Exception e) {
			e.printStackTrace();
		}

	}

	static void Print(AddressBook addressBook) {
		for (Person person : addressBook.getPersonList()) {
			System.out.println("Person ID: " + person.getId());
			System.out.println("  Name: " + person.getName());
			if (person.hasEmail()) {
				System.out.println("  E-mail address: " + person.getEmail());
			}

			for (Person.PhoneNumber phoneNumber : person.getPhoneList()) {
				switch (phoneNumber.getType()) {
				case MOBILE:
					System.out.print("  Mobile phone #: ");
					break;
				case HOME:
					System.out.print("  Home phone #: ");
					break;
				case WORK:
					System.out.print("  Work phone #: ");
					break;
				}
				System.out.println(phoneNumber.getNumber());
			}
		}
	}
}
  ```
  
### proto文件简单语法分析
参考Google开发文档(https://developers.google.com/protocol-buffers/docs/proto)

> .proto文件起始于一个包声明，这能够帮助在不同的工程中避免命名冲突。在使用Java的情况下，package name被用来作为Java的package，除非你详细指定了一个java_package，正如我们在这里是这样做的。即使你却是提供了一个java_package，你也应该仍旧定义一个正常的package以便在Protocol Buffers命名空间和非Java语言中避免命名冲突。

> 在包声明以后，你会看到两个Java特有的配置项：java_package和java_outer_classname。java_package指定你生成的类要存在于什么包下。如果你没有具体的指定，它会简单的与package给出的package name相匹配，但是这些名字通常不适合做为Java包的名字（因为它们通常不以域名开头）。java_outer_classname配置项定义了那些在这个文件中包含所有类的名称。如果你没有具体指定java_outer_classname，它将会转换文件名为驼峰式来产生。例如，“my_proto.proto”将默认使用“MyProto”做为outer class name(类的文件名)。

> 接下来是你的message定义。一个message只是一系列具有类型的域的集合。许多标准的简单类型可以做为可用的域类型，包括bool, int32, float, double, 和string。你同样可以向你的message中添加更多的结构，通过把其他的message类型当做域类型使用－－在上面的例子中，Person message包含PhoneNumber message，而AddressBook message包含Person message。你甚至可以通过内部嵌套其它message的方式定义message，你可以看到，PhoneNumber类型被定义在Person里面。你同样可以定义enum类型，如果你希望其中的一个你的域拥有预定义列表中的一个值－－在这里，你希望指定一个phone number的值是MOBILE, HOME, 或者WORK中的一个。

> 这里在每个元素上面的“=1”，“=2”标记，辨识唯一的“tag”，这些“tag”是域在二进制编码的时候使用的。Tag编号从1到15，需要比更大的数字少一个字节，因此，出于优化考虑你可以决定使用常用的或者重复的元素。而从16到更高的编号留给不常用的可选元素。在重复域中的每一个元素都需要重新编码tag number，所以，使用重复域对优化来说是特别好的选择。

> 每一个域都必须被下面之一的modifer注解：
* required：域的值必须被提供，否则message会被认为是“未初始化的”。试图创建一个未初始化的message将会抛出一个RuntimeException。转化一个未初始化的message将会抛出一个IOException。除了这些，required域的行为和optional域表现相同。
* optional：该域可能被设置，也可能未被设置。如果一个optional域未被设置，默认的值将会被使用。对于简单类型来说，你可以指定你自己的默认值，就像我们在例子中对phone number类型所做的那样。否则，一个系统默认的值将被使用：数值类型是0，字符串类型是""，布尔类型是false。对于嵌入式的message来说，默认值总是这个message的没有任何域被设置过的“默认实例”或“原型”。调用访问器去获取一个optional或者required域的值，这些域如果没有被具体设置，那么总是返回域的默认值。
* repeated：域将会被重复任意次数（包括0次）。重复的值的顺序将会被保存在protocol buffer中。可以把重复的域想成变长的数组。 
  
  
### 总结
最近查看一个开源p2p项目的代码，其中指令协议本身是已经固定好的，十分适合使用Protocol Buffer作为协议命令传输。
