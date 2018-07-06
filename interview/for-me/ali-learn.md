## 1. 给一个文件路径，返回一个list, 根据换行进行分割
~~~
list<String> fun(String file);
~~~

~~~java
public class BufferedInputFile {
    public static void main(String[] args) throws Exception {
        read("D:\\DirectoryDemo.java");
    }
    public static List<String> read(String filename) throws Exception{
        String s;
        List<String>  list = new ArrayList<String>();
        BufferedReader in = new BufferedReader(new FileReader(filename));
        while((s = in.readLine()) != null){
        	list.add(s);
        }
        in.close();
        return list;
    }
    
    
     public static String readFile(String filename) throws Exception{
            String s;
            StringBuilder sb = new StringBuilder();
            BufferedReader in = new BufferedReader(new FileReader(filename));
            while((s = in.readLine()) != null){
            	sb.append(s + "\n");
            }
            in.close();
            return sb.toString();
        }
}
~~~



## 2. 
~~~java
class node{
    String name;
    float score;
}
~~~~
对list中的node 按照score 从小到大排列.
~~~
List<node> fun(List<node> list)
~~~

~~~java
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class NodeSort {

	public static void main(String[] args) {
		Node n1 = new Node(2.3f, "node1");
		Node n2 = new Node(1.0f, "node2");
		Node n3 = new Node(10.3f, "node3");
		Node n4 = new Node(4.5f, "node4");
		List<Node> list = new ArrayList<>();
		List<Node> list2 = new ArrayList<>();
		list.add(n1);
		list.add(n2);
		list.add(n3);
		list.add(n4);
		
		list2 = sort(list);
		for (Node node : list2) {
			System.out.println(node.getScore());
		}
	}


	@SuppressWarnings("unchecked")
	public static List<Node>  sort(List<Node> list){
		Collections.sort(list, new ScoreSort());
		return list;
	}
	
	public static class ScoreSort implements Comparator{
		@Override
		public int compare(Object o1, Object o2) {
			Node n1 = (Node) o1;
			Node n2 = (Node) o2;
			return n1.getScore().compareTo(n2.getScore());
		}
	}
}
class Node{
	private Float score;
	private String name;

	public Float getScore() {
		return score;
	}
	public void setScore(Float score) {
		this.score = score;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Node(Float score, String name) {
		super();
		this.score = score;
		this.name = name;
	}
	public Node() {
		super();
	}
}

~~~



## 3. 有这样一种类型的句子，
我想听一首个叫@{song} 的歌/给我来一首@{song}

我想听一首叫@{青花瓷}的歌

要求从这个句子中提取出song
~~~
String fun(String query){}
~~~

~~~java
public class SongName {
	public static void main(String[] args) {
		String query = "我想听一首叫@{青花瓷}的歌";
		String name = getSongName(query);
		System.out.print(name);

	}

	public static String getSongName(String query) {
	            String group1 = "";
		        String pattern  = "@\\{(.*)}";
		        String pattern1 = "@\\{(\\w*)}";
		        String pattern2 = "@\\{([\\u4e00-\\u9fa5]*)}";
                Pattern compile = Pattern.compile(pattern, Pattern.UNICODE_CHARACTER_CLASS);
                Matcher match = compile.matcher(query);
                while(match.find()){
                    group1 = match.group(1);
                }
                return group1;
	}
}



~~~