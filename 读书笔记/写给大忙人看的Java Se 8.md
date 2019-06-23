### 第一章Lambda练习：
2、使用java.io.File类的listFiles(FileFilter)和isDirectory方法，编写一个返回指定目录下所有子目录的方法。使用lambda表达式来代替FileFilter对象，再将它改写为一个方法的引用。
```java
	private static void listFiles(String path) {
		File file = new File(path);
		File[] dirs = file.listFiles((_file) -> _file.isDirectory());
		Arrays.stream(dirs).forEach(System.out::println);
	}
```
3、使用java.io.File类的list(FilenameFilter)方法，编写一个返回指定目录下、具有指定扩展名的所有文件。使用lambda表达式(而不是FilenameFilter)来实现。他会捕获闭合作用域中的哪些变量？
```java
	private static void listFiles(String path, String extName) {
		File file = new File(path);
		File[] dirs = file.listFiles((_file, _name) -> _name.endsWith(extName));
		Arrays.stream(dirs).forEach(System.out::println);
	}
```
7、编写一个静态方法andThen,它接受两个Runnable实例作为参数，并返回一个分别运行这两个实例的Runnable对象。在main方法中，向andThen方法传递两个lambda表达式，并运行返回的实例。
```java
	public static void main(String[] args) {
		Runnable task = andThen(() -> System.out.println("Hello"), () -> System.out.println("World"));
		new Thread(task).start();
	}

	public static Runnable andThen(Runnable task1, Runnable task2) {
		return () -> {
			task1.run();
			task2.run();
		};
	}
```
8、当一个lambda表达式捕获了如下增强for循环中的值时，会发生什么？这样做是否合法？每个lambda表达式都捕获了一个不同的值，还是他们都获得了最终的值？如果使用传统的for循环，例如for (int i=0;i<names.length;i++),又会发生什么？

```java
	public static void main(String[] args) {
		String[] names = { "Peter", "Paul", "Mary" };
		List<Runnable> runners = new ArrayList<>();
		// 可以
		for (String name : names) {
			runners.add(() -> System.out.println(name));
		}
		// 可以
		for (int i = 0; i < names.length; i++) {
			String name = names[i];
			runners.add(() -> System.out.println(name));
			// 直接调用下面这句是不行的
			// Local variable i defined in an enclosing
			// scope must be final or effectively final
			// runners.add(() -> System.out.println(names[i]));
		}

		for (int i = 0; i < runners.size(); i++) {
			new Thread(runners.get(i)).start();
		}
	}
```
### 第二章Stream API练习：
wordcount
```java
	public static void wordCount() throws IOException {
		// 读入每一行
		List<String> allLines = Files.readAllLines(Paths.get("/Users/admin/Desktop/in.txt"));

		Files.lines(Paths.get("/Users/admin/Desktop/in.txt")).flatMap(line -> Arrays.stream(line.split(" ")))
				.collect(Collectors.groupingBy(word -> word, Collectors.counting()));
		HashMap<String, Integer> counter1 = new HashMap<>();
		allLines.stream().flatMap(Line -> Arrays.stream(Line.split(" "))).forEach(word -> {
			Integer count = counter1.get(word);
			counter1.put(word, count == null ? 1 : count + 1);
		});
		counter1.forEach((k, v) -> System.out.println(k + ":" + v));

		// 读入为String
		HashMap<String, Integer> counter2 = new HashMap<>();
		String content = new String(Files.readAllBytes(Paths.get("/Users/admin/Desktop/in.txt")));
		Arrays.stream(content.split(" ")).forEach(word -> {
			Integer count = counter2.get(word);
			counter2.put(word, count == null ? 1 : count + 1);
		});
		counter2.forEach((k, v) -> System.out.println(k + ":" + v));

		// 使用groupingBy
		Map<String, Long> count = allLines.stream().map(line -> line.split(" ")).flatMap(Arrays::stream)
				.collect(Collectors.groupingBy(word -> word, Collectors.counting()));
		count.forEach((k, v) -> System.out.println(k + ":" + v));

		// 使用Files.lines
		Map<String, Long> count2 = Files.lines(Paths.get("/Users/admin/Desktop/in.txt"))
				.flatMap(line -> Arrays.stream(line.split(" ")))
				.collect(Collectors.groupingBy(word -> word, Collectors.counting()));
		count2.forEach((k, v) -> System.out.println(k + ":" + v));
	}
```