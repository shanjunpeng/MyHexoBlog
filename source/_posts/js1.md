title: js-创建对象&继承

date: 2015-06-18 09:53:11

tags:
- javascript

categories: javascript

---

##一、创建对象

###1、工厂模式
工厂模式可以解决多个相似对象的问题，却无法解决对象识别的问题（不知道对象的类型）  
<!--more--> 

    function createPerson(name,age){
    	var o = new Object();
		o.name=name;
		o.age=age;
		o.sayName = function(){
			alert(this.name);		
		}
		return o;
    }
	var person = createPerson("Tom",25);

###2、构造函数模式
缺点：它的每个成员都无法得到复用，包括函数。将函数放到全局，又会失去封装性。  

	function Person(name,age){
		this.name=name;
		this.age=age;
		//把函数放到外部，防止每创建一个对象就创建一个sayName函数的对象
		this.sayName = sayName;
	}
	function sayName(){
		alert(this.name);
	}
	var person = new Person("Jim",18);

###3、原型模式
原型模式省略了构造函数传递初始化参数这一环节，结果所有实例在默认情况下都将取得相同的属性值。  
原型模式中所有属性是被多实例共享的，如果是引用类型（如数组），其中一个实例修改了数组内容，其他实例也会反映出这个修改。  

	function Person(){
	}
	Person.prototype.name="Jim";
	Person.prototype.age=18;
	Person.prototype.friends = ["Lily","Tom"];
	Person.prototype.sayName = function(){
		alert(this.name);
	}

原型模式字面量写法：  

	Person.prototype={
		name:"Jim",
		age:18,
		sayName:function(){
			alert(this.name);
		}	
	}

###4、组合使用构造函数模式和原型模式
使用最广泛，认同度最高的创建自定义类型的方式。  
使用构造函数定义实例属性，使用原型定义共享的属性和方法。  

	function Person(name,age){
		this.name = name;
		this.age=age;
		this.hobby=["game","spot"];		
	}
	Person.prototype = {
		constructor:Person,
		sayName:function(){
			alert(this.name);
		}
	}

###5、动态原型模式
把所有信息封装在构造函数之中，在构造函数中初始化原型。  

	function Person(name,age){
		this.name=name;
		this.age=age;
		//只需要在第一次调用构造函数时初始化原型
		if(typeof this.sayName!="function"){
			Person.prototype.sayName = function(){
				alert(this.name);
			}
			Person.prototype.sayAge = function(){
				alert(this.age);
			}
		}
	}

###6、寄生构造函数模式
可以在特殊情况下用来给对象创建构造函数，如创建一个具有额外方法的特殊数组。  

	function SpecialArray(){
		var arr = new Array();
		arr.push.apply(arr,arguments);
		arr.joinString = function(){
			return this.join("|");
		};
		return arr;
	}
	var colors = new SpecialArray("red","yellow");
	alert(colors.joinString());

---
##二、继承
###1、原型链
让子类的原型对象等于父类的一个实例。

	//父类
	function SuperType(){
		this.property = 1;
		this.colors = ["blue","red"];	
	}
	SuperType.prototype.getSuperValue=function(){
		return this.property;
	}
	//子类
	function SubType(){
		this.subProp = 2;
	}
	//使用父类的实例作为子类的prototype
	SubType.prototype=new SuperType();

	SubType.prototype.getSubValue=function(){
		return this.subProp;
	}
	//可以重写父类方法
	SubType.prototype.getSuperValue=function(){
		return 3;
	}
	
缺点：子类实例引用属性的修改会影响其他实例  

	var instance = new SubType();
	var instance2 = new SubType();
	instance.colors.push("black");
	alert(instance.colors);//blue,red,black
	alert(instance2.colors);//blue,red,black


###2、寄生组合式继承
寄生组合继承是引用类型最理想的继承范式，是实现基于类型继承的最有效方式。

	function inheritPrototype(subType,superType){
		var prototype = Object.create(superType.prototype);//创建对象
		prototype.constructor=subType;
		subType.protoType=prototype;//指定对象
	}
	//父类
	function SuperType(name){
		this.colors=["red","blue"];
		this.name=name;
	}
	SuperType.prototype={
		flag:true,
		sayName:function(){
			alert(this.name);
		}
	};
	//子类
	function SubType(name,age){
		//继承属性
		SuperType.call(this,name);
		this.age=age;
	}
	//继承prototype
	inheritPrototype(SubType,SuperType);
	
	SubType.prototype.sayAge=function(){
		alert(this.age);
	};
	
	var subtype = new SubType("sjp",25);
	var subtype2 = new SubType("xuxu",22);
	//继承了方法
	subtype.sayName();//sjp
	//继承了属性
	alert(subtype.flag);//true
	//子类实例属性的修改不影响其他实例
	subtype.colors.push("black");
	alert(subtype2.colors);//blue,red,yellow
