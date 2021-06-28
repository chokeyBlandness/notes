# SecureRandom类序列化问题

**SecureRandom cannot be serialized, even though it implements Serializable**

### Description

SecureRandom extends java.util.Random, which implements Serializable.Hence, SecureRandom is serializable, too.

However,when some code tries to serialize SecureRandom,it tries to serialize all its "member" objects.One of the member objects is a MessageDigest.That is not serializable.So the code can't serialize SecureRandom, since one of its member objects is not serializable.

### BugInfo

java.io.NotSerializableException: java.security.MessageDigest$Delegate

### example

~~~java
import java.io.*;
import java.security.*;

public class Random {
    public static void main(String[] args) {
		try {
		SecureRandom random = new SecureRandom();
		FileOutputStream ostream = new FileOutputStream("t.tmp");
		ObjectOutputStream p = new ObjectOutputStream(ostream);

		p.writeObject(random);

		p.flush();
		ostream.close();
		} catch (Exception e) {
			System.out.println(e);
		}
    }
}
~~~

