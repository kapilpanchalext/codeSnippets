# Java 8 and above Streams Code Snippet [Reference]

```java
import java.util.Arrays;
import java.util.Comparator;
import java.util.DoubleSummaryStatistics;
import java.util.List;
import java.util.Map;
import java.util.NoSuchElementException;
import java.util.Optional;
import java.util.Set;
import java.util.Vector;
import java.util.function.BinaryOperator;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class SpringStream {

	public static void main(String[] args) {
		
		Employee[] arrayOfEmps = {
				new Employee(1, "Jeff Bezos", 100000),
				new Employee(2, "Bill Gates", 200000),
				new Employee(3, "Mark Zuckerburg", 300000)
		};
		
		List<Employee> empList = Arrays.asList(arrayOfEmps);
		System.err.println(empList);
		empList.stream().forEach(element -> element.salaryIncrement(element, 10L));
		System.err.println(empList);
		
		Integer[] empIds = {1, 2, 3};
		List<Employee> employees = Stream.of(empIds)
				.map(element -> {
					Employee emp = new Employee();					
					emp.setEmpId(element);
					return emp;
				}).collect(Collectors.toList());
		System.err.println("EMPLOYEES: " + employees);
				
		List<Employee> employees1 = Stream.of(arrayOfEmps)
				.filter(element -> element != null)
				.filter(element -> element.getSalary() > 200000L)				
				.collect(Collectors.toList());
		System.err.println("EMPLOYEES 1: " + employees1);
		
		List<String> firstNames = Stream.of(arrayOfEmps)
                .flatMap(employee -> Stream.of(employee.getName().substring(0, employee.getName().indexOf(" "))))
                .peek(element -> System.err.println("Element: " + element))
                .map(String::trim)
                .collect(Collectors.toList());
		
		System.err.println("FLAT MAP OPERATIONS: " + firstNames);
		
		/*INFINITE STREAM*/
		Stream<Integer> infiniteStream = Stream.iterate(2, i -> i * 2);
		List<Integer> collect = infiniteStream
			      .skip(0)
			      .limit(10)
			      .collect(Collectors.toList());
		System.err.println("INFINITE STREAM: " + collect);
    
	    Employee employees2 = Stream.of(arrayOfEmps)
				.filter(element -> element != null)
				.filter(element -> element.getSalary() > 200000L)				
				.findFirst()
			    .orElse(null);
		System.err.println("EMPLOYEES 2: " + employees2);
		
		/*SORTED ARRAYS*/
		List<Employee> employeesSorted = empList.stream().sorted((e1,e2) -> e1.getName().compareTo(e2.getName())).collect(Collectors.toList());				
		System.err.println("SORTED EMPLOYEES: " + employeesSorted);
		
		/*MIN*/
		Employee employeesMin = empList.stream().min((e1,e2) -> e1.getEmpId()-e2.getEmpId()).orElseThrow(NoSuchElementException::new);
		System.err.println("MIN EMPLOYEE: " + employeesMin);
		
		/*MAX*/
		Employee employeesMax = empList.stream().max(Comparator.comparing(Employee::getSalary)).orElseThrow(NoSuchElementException::new);
		System.err.println("MAX SALARY: " + employeesMax);
		
		/*allMatch anyMatch noneMatch*/
		List<Integer> intList = Arrays.asList(2, 4, 5, 8);
		boolean allEven = intList.stream().allMatch(i -> i % 2 == 0);
	    boolean oneEven = intList.stream().anyMatch(i -> i % 2 == 0);
	    boolean noneMultipleOfThree = intList.stream().noneMatch(i -> i % 3 == 0);
	    
	    System.err.println("ALL EVEN: " + allEven);
	    System.err.println("ONE EVEN: " +  oneEven);
	    System.err.println("NONE MULTPILE OF THREE: " + noneMultipleOfThree);
	    
	    /*mapToInt*/
	    Integer latestEmpId = empList.stream()
	    	      .mapToInt(Employee::getEmpId)
	    	      .max()
	    	      .orElseThrow(NoSuchElementException::new);
	    System.err.println("LATEST EMP ID: " + latestEmpId);
	    
	    /*average*/
	    Double avgSal = empList.stream()
	    	      .mapToDouble(Employee::getSalary)
	    	      .average()
	    	      .orElseThrow(NoSuchElementException::new);
	    System.err.println("AVERAGE SALARY: " + avgSal);
	    
	    /*REDUCTION OPERATIONS*/
	    Long sumSal = empList.stream()
	    	      .map(Employee::getSalary)
	    	      .reduce(0L, Long::sum);
	    System.err.println("SUM OF SALARY: " + sumSal);
	    
	    /*EMP NAMES JOINING OPERATIONS*/
	    String empNames = empList.stream()
	    	      .map(Employee::getName)
	    	      .collect(Collectors.joining(", "))
	    	      .toString();
	    System.err.println("EMP NAMES: " + empNames);
	    
	    /*toSet()*/
	    Set<String> empNamesSet = empList.stream()
	            .map(Employee::getName)
	            .collect(Collectors.toSet());
	    System.err.println("SET STRING: " + empNamesSet);
	    
	    /*vector - Collection*/
	    Vector<String> empNamesVector = empList.stream()
	            .map(Employee::getName)
	            .collect(Collectors.toCollection(Vector::new));
	    System.err.println("Vector: " + empNamesVector);
	    
	    /*summerising double*/
	    DoubleSummaryStatistics stats = empList.stream()
	    	      .collect(Collectors.summarizingDouble(Employee::getSalary));
	    System.err.println("DOUBLE SUMMARY STATISTICS: " + stats);
	    
	    /*summary statistics*/
	    DoubleSummaryStatistics summaryStats = empList.stream()
	    	      .mapToDouble(Employee::getSalary)
	    	      .summaryStatistics();

	    System.err.println("SUMMARY STATISTICS: " + summaryStats);
	    
	    /*partitioning by - splits into two groups*/
	    List<Integer> intListPartition = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
	    Map<Boolean, List<Integer>> isEven = intListPartition.stream().collect(
	      Collectors.partitioningBy(i -> i % 2 == 0));
	    System.err.println("INT LIST IS EVEN: " + isEven);
	    
	    /*grouping by - splits into more than two groups*/
	    Map<Character, List<Employee>> groupByAlphabet = empList.stream().collect(
	    	      Collectors.groupingBy(e -> new Character(e.getName().charAt(0))));
	    System.err.println("GROUP BY ALPHABET: " + groupByAlphabet);
	    
	    /*mapping - splits into more than two groups also with another data type*/
	    Map<Character, List<Integer>> idGroupedByAlphabet = empList.stream().collect(
	    	      Collectors.groupingBy(e -> new Character(e.getName().charAt(0)),
	    	        Collectors.mapping(Employee::getEmpId, Collectors.toList())));
	    System.err.println("ID GROUPED BY ALPHABET: " + idGroupedByAlphabet);
	    
	    /*reducing ()*/
	    Double percentage = 10.0;
	    Double salIncrOverhead = empList.stream().collect(Collectors.reducing(
	        0.0, e -> e.getSalary() * percentage / 100, (s1, s2) -> s1 + s2));
	    
	    System.err.println("SAL INCR OVERHEAD: " + salIncrOverhead);
	    
	    /*reducing and groupingBy together*/
	    Comparator<Employee> byNameLength = Comparator.comparing(Employee::getName);
	    
	    Map<Character, Optional<Employee>> longestNameByAlphabet = empList.stream().collect(
	      Collectors.groupingBy(e -> new Character(e.getName().charAt(0)),
	        Collectors.reducing(BinaryOperator.maxBy(byNameLength))));
	    
	    System.err.println("LONGEST NAME BY ALPHABET: " + longestNameByAlphabet);
	    
	    /*PARALLEL STREAMS*/
	    empList.stream().parallel().forEach(element -> element.salaryIncrement(element, 10L));
	    System.err.println("PARALLEL STREAM: " + empList);
	    
	    /*INFINITE STREAMS*/
	    Stream.generate(Math::random)
	      .limit(5)
	      .forEach(System.err::println);
	    
	    /*ITERATE*/
	    Stream<Integer> evenNumStream = Stream.iterate(2, i -> i * 2);

	    List<Integer> collectIterate = evenNumStream
	      .limit(5)
	      .collect(Collectors.toList());
	    
	    System.err.println("COLLECT ITERATE: " + collectIterate);
	    
	    /*take while - While certain conditions are true*/
	    Stream.iterate(1, i -> i + 1)
        .takeWhile(n -> n <= 10)
        .map(x -> x * x)
        .forEach(System.err::println);
	    System.err.println();
	    
	    /*difference between takeWhile and filter*/
	    Stream.of(1,2,3,4,5,6,7,8,9,0,9,8,7,6,5,4,3,2,1,0)
        .takeWhile(x -> x <= 5)
        .forEach(System.err::println);
	    
	    System.err.println();
	    
	    Stream.of(1,2,3,4,5,6,7,8,9,0,9,8,7,6,5,4,3,2,1,0)
        .filter(x -> x <= 5)
        .forEach(System.err::println);
	    System.err.println();
	    
	    /*drop while is reverse of takeWhile*/
	    Stream.of(1,2,3,4,5,6,7,8,9,0,9,8,7,6,5,4,3,2,1,0)
        .dropWhile(x -> x <= 5)
        .forEach(System.err::println);
	    System.err.println();
	    
	    /*iterate java9*/
	    Stream.
		iterate(1, i -> i < 256, i -> i * 2)
		.forEach(System.err::println);
	    
	    System.err.println();
	    /*nullable*/
	    Integer[] number = {1,2,3,4,5,6,7,8,9,0,9,8,7,6,5,4,3,2,1,0};
	    Stream<Integer> result = number != null
	            ? Stream.of(number)
	            : Stream.empty();
	    result.forEach(e -> System.err.println("Stream Element: " + e));
	    
	    
	}
}

class Employee {
	
	private int empId;
	private String name;
	private long salary;
	
	public Employee(int empId, String name, long salary) {
		this.empId = empId;
		this.name = name;
		this.salary = salary;
	}

	public Employee salaryIncrement(Employee element, long d) {
		element.setSalary(element.getSalary()+d);
		return element;
	}

	public Employee() {
		
	}

	public int getEmpId() {
		return empId;
	}

	public void setEmpId(int empId) {
		this.empId = empId;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public long getSalary() {
		return salary;
	}

	public void setSalary(long salary) {
		this.salary = salary;
	}

	@Override
	public String toString() {
		return "Employee [empId=" + empId + ", name=" + name + ", salary=" + salary + "]";
	}	
}
```
[Reference]: (https://stackify.com/streams-guide-java-8/)
