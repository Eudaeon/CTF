# Up a Stream

## Recon

JAR with output file.

## Reversing

```java
public class Challenge {
    public static void main(String[] strArr) {
        Stream<String> stream = dunkTheFlag("FLAG").stream();
        PrintStream printStream = System.out;
        Objects.requireNonNull(printStream);
        stream.forEach(printStream::println);
    }

    private static List<String> dunkTheFlag(String str) {
        return Arrays.asList(((String) ((List) ((String) ((List) ((String) ((List) str.chars().mapToObj(i -> {
            return Character.valueOf((char) i);
        }).collect(Collectors.toList())).stream().peek(ch -> {
            hydrate(ch);
        }).map(ch2 -> {
            return ch2.toString();
        }).reduce("", (str2, str3) -> {
            return str3 + str2;
        })).chars().mapToObj(i2 -> {
            return Character.valueOf((char) i2);
        }).collect(Collectors.toList())).stream().map(ch3 -> {
            return ch3.toString();
        }).reduce((v0, v1) -> {
            return v0.concat(v1);
        }).get()).chars().mapToObj(i3 -> {
            return Integer.valueOf(i3);
        }).collect(Collectors.toList())).stream().map(num -> {
            return moisten(num.intValue());
        }).map(num2 -> {
            return Integer.valueOf(num2.intValue());
        }).map(Challenge::drench).peek(Challenge::waterlog).map(Challenge::dilute).map((v0) -> {
            return Integer.toHexString(v0);
        }).reduce("", (str4, str5) -> {
            return str4 + str5 + "O";
        })).repeat(5));
    }

    public static Integer hydrate(Character ch) {
        return Integer.valueOf(ch.charValue() - 1);
    }

    public static Integer moisten(int i) {
        return Integer.valueOf((int) (i % 2 == 0 ? i : Math.pow(i, 2.0d)));
    }

    private static Integer drench(Integer num) {
        return Integer.valueOf(num.intValue() << 1);
    }

    private static Integer dilute(Integer num) {
        return Integer.valueOf((num.intValue() / 2) + num.intValue());
    }

    private static byte waterlog(Integer num) {
        return Integer.valueOf(((((num.intValue() + 2) * 4) % 87) ^ 3) == 17362 ? num.intValue() * 2 : num.intValue() / 2).byteValue();
    }
}
```

Uses streams to process input. `waterlog` not necessary to reverse, because app calls it with `peek`, which doesn't modify input.

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class Solver {
    private static final String output = "b71bO12cO156O6e43Od8O69c3O5cd3O144Oe4O6e43O37cbOf6O69c3O1e7bO156O3183O69c3O6cO8b3bOc0O1e7bO156OfcO50bbO69c3Oc0O102O6e43OdeOb14bOc6OfcOd8Ob71bO12cO156O6e43Od8O69c3O5cd3O144Oe4O6e43O37cbOf6O69c3O1e7bO156O3183O69c3O6cO8b3bOc0O1e7bO156OfcO50bbO69c3Oc0O102O6e43OdeOb14bOc6OfcOd8Ob71bO12cO156O6e43Od8O69c3O5cd3O144Oe4O6e43O37cbOf6O69c3O1e7bO156O3183O69c3O6cO8b3bOc0O1e7bO156OfcO50bbO69c3Oc0O102O6e43OdeOb14bOc6OfcOd8Ob71bO12cO156O6e43Od8O69c3O5cd3O144Oe4O6e43O37cbOf6O69c3O1e7bO156O3183O69c3O6cO8b3bOc0O1e7bO156OfcO50bbO69c3Oc0O102O6e43OdeOb14bOc6OfcOd8Ob71bO12cO156O6e43Od8O69c3O5cd3O144Oe4O6e43O37cbOf6O69c3O1e7bO156O3183O69c3O6cO8b3bOc0O1e7bO156OfcO50bbO69c3Oc0O102O6e43OdeOb14bOc6OfcOd8O";

    public static void main(String[] args) {
        String recovered = reverseOutput(output);
        System.out.println(recovered);
    }

    private static String reverseOutput(String output) {
        List<String> hexValues = Arrays.stream(output.split("O"))
            .map(String::strip)
            .filter(s -> !s.isEmpty())
            .collect(Collectors.toList());

        int blockSize = hexValues.size() / 5;

        List<String> firstBlock = hexValues.subList(0, blockSize);
        
        String reversed = firstBlock.stream()
            .map(s -> Integer.parseInt(s, 16))
            .map(Solver::undilute)
            .map(Solver::undrench)
            .map(Solver::demoisten)
            .map(i -> (char) i.intValue())
            .map(String::valueOf)
            .collect(Collectors.joining());

        return new StringBuilder(reversed).reverse().toString();
    }
    
    private static int demoisten(int i) {
        return (i % 2 != 0) ? (int) Math.sqrt(i) : i;
    }
    
    private static int undrench(int number) {
        return number >> 1;
    }
    
    private static int undilute(int number) {
        return (2 * number) / 3;
    }
}
```

```plaintext
HTB{JaV@_STr3@m$_Ar3_REaLlY_Hard}
```
