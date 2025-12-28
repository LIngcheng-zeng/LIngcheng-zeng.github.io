```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

/**
 * G6 节点数据结构
 */
@Data
@AllArgsConstructor
class G6Node {
    private String id;
    private String label;
    private Object data; // 存储原始数据

    public G6Node(String id, String label) {
        this.id = id;
        this.label = label;
    }
}

/**
 * G6 边数据结构
 */
@Data
@AllArgsConstructor
class G6Edge {
    private String source;
    private String target;
}

/**
 * G6 图数据结构
 */
@Data
@NoArgsConstructor
class G6GraphData {
    private List<G6Node> nodes;
    private List<G6Edge> edges;
}

```

```java
public interface LabelValue {
    String getLabel();
}
```

```java

public interface SonAndParentKey<T extends LabelValue> {
    String getSelfKey();

    String getParentKey();

    T getSelfData();
}

```

```java
import java.util.*;
import java.util.stream.Collectors;

public class VisualTreeUtil {
    /**
     * 将实现了SonAndParentKey接口的列表转换为G6树形结构数据
     *
     * @param dataList 源数据列表
     * @param <T>      数据类型
     * @return G6树形结构数据，包含nodes和edges
     */
    public static <T extends LabelValue> Map<String, Object> convertToG6Tree(List<? extends SonAndParentKey<T>> dataList) {
        if (dataList == null || dataList.isEmpty()) {
            return createEmptyTree();
        }

        Map<String, Set<String>> parentMapSonList = dataList.stream()
                .collect(Collectors.groupingBy(SonAndParentKey::getParentKey,
                        Collectors.mapping(SonAndParentKey::getSelfKey, Collectors.toSet())));
        // 构建节点和边的数据
        List<Map<String, Object>> nodes = new ArrayList<>();
        List<Map<String, Object>> edges = new ArrayList<>();

        // 使用Map快速查找节点
        Map<String, SonAndParentKey<T>> nodeMap = new HashMap<>();

        // 首先收集所有节点信息
        for (SonAndParentKey<T> item : dataList) {
            nodeMap.put(item.getSelfKey(), item);
        }

        // 创建节点数据
        for (SonAndParentKey<T> item : dataList) {
            Map<String, Object> node = new HashMap<>();
            node.put("id", item.getSelfKey());
            node.put("children", parentMapSonList.getOrDefault(item.getSelfKey(), Collections.emptySet())); // 可根据需要调整标签显示内容
            // 如果有自定义数据，可以添加到节点中
            if (item.getSelfData() != null) {
                node.put("data", JsonUtils.toJson(item.getSelfData()));
            }
            nodes.add(node);
        }

        // 创建边数据
        for (SonAndParentKey<T> item : dataList) {
            String parentKey = item.getParentKey();
            String selfKey = item.getSelfKey();

            // 如果有父节点且父节点存在于数据列表中
            if (parentKey != null && !parentKey.isEmpty() && nodeMap.containsKey(parentKey)) {
                Map<String, Object> edge = new HashMap<>();
                edge.put("source", parentKey);
                edge.put("target", selfKey);
                edges.add(edge);
            }
        }

        // 组装返回结果
        Map<String, Object> result = new HashMap<>();
        result.put("nodes", nodes);
        result.put("edges", edges);

        return result;
    }


    /**
     * 创建空树结构
     */
    private static Map<String, Object> createEmptyTree() {
        Map<String, Object> result = new HashMap<>();
        result.put("nodes", new ArrayList<>());
        result.put("edges", new ArrayList<>());
        return result;
    }

    public static void main(String[] args) {
        // 示例使用
        // 创建测试数据
        List<TestSonAndParent> testData = new ArrayList<>();
        testData.add(new TestSonAndParent("1", "0", "Root"));
        testData.add(new TestSonAndParent("2", "1", "Child1"));
        testData.add(new TestSonAndParent("3", "1", "Child2"));
        testData.add(new TestSonAndParent("4", "2", "GrandChild1"));
        testData.add(new TestSonAndParent("5", "2", "GrandChild2"));

        // 转换为G6树形结构
        Map<String, Object> treeData = convertToG6Tree(testData);

        System.out.println("Nodes: " + treeData.get("nodes"));
        System.out.println("Edges: " + treeData.get("edges"));
        System.out.println(treeData.toString().replaceAll("=", ":").replaceAll("\"", "\'"));
    }
}

class StringLabelValue implements LabelValue {
    private String label;

    public StringLabelValue(String label) {
        this.label = label;
    }

    @Override
    public String getLabel() {
        return label;
    }
}

// 测试用的实现类
class TestSonAndParent implements SonAndParentKey<StringLabelValue> {
    private String selfKey;
    private String parentKey;
    private StringLabelValue selfData;

    public TestSonAndParent(String selfKey, String parentKey, String selfData) {
        this.selfKey = selfKey;
        this.parentKey = parentKey;
        this.selfData = new StringLabelValue(selfData);
    }

    @Override
    public String getSelfKey() {
        return selfKey;
    }

    @Override
    public String getParentKey() {
        return parentKey;
    }

    @Override
    public StringLabelValue getSelfData() {
        return selfData;
    }
}

```



