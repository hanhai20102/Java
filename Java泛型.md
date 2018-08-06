# java泛型

## 集合中的泛型
PS:
```java
import java.util.ArrayList;
 
import charactor.APHero;
 
public class TestGeneric {
 
    public static void main(String[] args) {
        ArrayList<APHero> heros = new ArrayList<APHero>();
        //只有APHero可以放进去----ADHero放不进去
        heros.add(new APHero());
        //获取的时候也不需要进行转型，因为取出来一定是APHero
        APHero apHero =  heros.get(0);
    }
}
```

**java7之后泛型可以简写**
>ArrayList<String> heros2 = new ArrayList<>();

## 支持泛型的类以及自定义泛型

```java
package po;

public class ReceiveEntity<T>{
	
    private Integer code;
    private String message;
    private T data;
    public Integer getCode() {
        return code;
    }
    public void setCode(Integer code) {
        this.code = code;
    }
    public String getMessage() {
        return message;
    }
    public void setMessage(String message) {
        this.message = message;
    }
    public T getData() {
        return data;
    }
    public void setData(T data) {
        this.data = data;
    }
    private ReceiveEntity() {
    }
    //对返回结果进行解析
    public static <T> T parseResult(ReceiveEntity<T> receivePo) {
    	Integer code = receivePo.getCode();
    	String message = null;
    	T data = null;
    	if(code==0){    //执行成功
        	data = receivePo.getData();
    	}else {
			message = receivePo.getMessage();
			System.out.println(message);
		}
		return data;
	}

}

```
+ 自定义泛型类就在定义类的后边添加<T>泛型标识符，在创建该类的对象的时候就传入具体的类型就可以了 例如ReceiveEntity<Integer> = new ReceiveEntity<>();
+ 在定义静态的方法时，因为静态方法不能访问类上的泛型因此需要在方法头定义使用的泛型列表。如上所示

## 泛型的转型

java对象可以实现向上转型，但是子类泛型不能转换为父类泛型

+ 假设泛型中存的是父类的类型，则可以传入父类以及子类的的类型，但是在取出的时候不知道具体的类型
+ 假设泛型中存的是子类的类型，只能存入对应的类的类型。
因此，子类的泛型如果转换为父类泛型，则此引用指向的是存放子类的集合，再存入父类的其他子类就会编译报错，因此子类泛型不能转换为父类泛型。

## 泛型的通配符
1. ? extends Hero 表示这是一个Hero子类的泛型
2. ? super Hero 表示这是一个Hero父类的泛型
3. ? 类型通配符 不主张使用没有意义。什么样的类型都可以存放，因此取出来的只能是Object类型

第一种情况 集合中存放的是Hero子类因此只能取不能存放。且取出的对象一定可以转换为Hero类型即向上转型
```java
public class TestGeneric {
   
    public static void main(String[] args) {
          
        ArrayList<APHero> apHeroList = new ArrayList<APHero>();
        apHeroList.add(new APHero());
         
        ArrayList<? extends Hero> heroList = apHeroList;
          
        //? extends Hero 表示这是一个Hero泛型的子类泛型
          
        //heroList 的泛型可以是Hero
        //heroList 的泛型可以使APHero
        //heroList 的泛型可以使ADHero
        //可以确凿的是，从heroList取出来的对象，一定是可以转型成Hero的
        Hero h= heroList.get(0);
        //但是，不能往里面放东西
        heroList.add(new ADHero()); //编译错误，因为heroList的泛型 有可能是APHero
          
    }
}
```
第二种情况 集合中存放的是Hero的父类，能取也能存，但是取出来的类型未知有风险，且如果取出来的Object类型（**Object类型不能强转为其他的类型**）
```java
public class TestGeneric {
    public static void main(String[] args) {
  
        ArrayList<? super Hero> heroList = new ArrayList<Object>();
        //? super Hero 表示 heroList的泛型是Hero或者其父类泛型
          
        //heroList 的泛型可以是Hero
        //heroList 的泛型可以是Object
        //所以就可以插入Hero
        heroList.add(new Hero());
        //也可以插入Hero的子类
        heroList.add(new APHero());
        heroList.add(new ADHero());
        //但是，不能从里面取数据出来,因为其泛型可能是Object,而Object是强转Hero会失败
        Hero h= heroList.get(0);
          
    }
  
}
```

## 使用Gson将json字符串转换成对应的实体类

### 使用泛型类：
###  一层泛型转换
```java
				ReceiveEntity<Integer> responsedata = gson.fromJson(str, new TypeToken<ReceiveEntity<Integer>>() {}.getType());

```

### 两层转换 通过具体的Class<T> clazz
使用如下的自定义工具包：
```java
package util;

import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;

import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.List;

import po.ReceiveEntity;


public class GsonUtils {

    public static <T> ReceiveEntity<T> fromJsonObject(String reader, Class<T> clazz) {
        Type type = new ParameterizedTypeImpl(ReceiveEntity.class, new Class[]{clazz});
        return new Gson().fromJson(reader, type);
    }

    public static <T> ReceiveEntity<List<T>> fromJsonArray(String reader, Class<T> clazz) {
        // 生成List<T> 中的 List<T>
        Type listType = new ParameterizedTypeImpl(List.class, new Class[]{clazz});
        // 根据List<T>生成完整的Result<List<T>>
        Type type = new ParameterizedTypeImpl(ReceiveEntity.class, new Type[]{listType});
        return new Gson().fromJson(reader, type);
    }

    public static class ParameterizedTypeImpl implements ParameterizedType {
        private final Class raw;
        private final Type[] args;

        public ParameterizedTypeImpl(Class raw, Type[] args) {
            this.raw = raw;
            this.args = args != null ? args : new Type[0];
        }

        @Override
        public Type[] getActualTypeArguments() {
            return args;
        }

        @Override
        public Type getRawType() {
            return raw;
        }

        @Override
        public Type getOwnerType() {
            return null;
        }
    }

    // 将Json数据解析成相应的映射对象
    public static <T> T parseJsonWithGson(String jsonData, Class<T> type) {
        Gson gson = new Gson();
        T result = gson.fromJson(jsonData, type);
        return result;
    }

    // 将Json数组解析成相应的映射对象列表
    public static <T> List<T> parseJsonArrayWithGson(String jsonData,
                                                     Class<T> type) {
        Gson gson = new Gson();
        List<T> result = gson.fromJson(jsonData, new TypeToken<List<T>>() {
        }.getType());
        return result;
    }
}

```
调用过程：传入具体的泛型类，先转化为List<T>的泛型，再把Json字符串转化为需要的实体类型。
```java
				ReceiveEntity<List<T>> responsedata = GsonUtils.fromJsonArray(str, clazz);

```
