---
id: 78
title: Using Builder Pattern in JUnit tests
date: 2012-12-11T09:24:00+00:00
author: gpanther
layout: post
guid: http://www.javaadvent.com/2012/12/using-builder-pattern-in-junit-tests/
permalink: /2012/12/using-builder-pattern-in-junit-tests.html
blogger_blog:
  - www.javaadvent.com
blogger_author:
  - Stefan Bulzan
blogger_permalink:
  - /2012/12/using-builder-pattern-in-junit-tests.html
blogger_internal:
  - /feeds/2481158163384033132/posts/default/2769130290548749220
dsq_thread_id:
  - 4962579208
categories:
  - 2021
  - builders
  - junit
  - testing
---
This is not intended to be a heavily technical post. The goal of this post is to give you some guidelines to make your JUnit testing life more easy, to enable you to write complex scenarios for tests in minutes with the bonus of having extremely readable tests.<br /><br />There are two major parts in a Unit tests that require writing a lot of bootstrap code:<br /><ul><li>the setup part: constructing your initial state requires building the initial objects that will be fed to your SUT (system under test)&nbsp;</li><li>the assertion part: constructing the desired image of your output objects and making assertions only on the needed data.</li></ul>In order to reduce the complexity of building objects for tests I suggest using the Builder pattern in the following&nbsp;interpretation:<br /><br />Here is the domain object:<br /><br /><pre style="border: solid thin gray;"><code><br />public class Employee {<br />    private int id;<br />    private String name;<br />    private Department department;<br /><br />    //setters, getters, hashCode, equals, toString methods   <br /></code></pre><br />The builder for this domain object will look like this:<br /><br /><pre style="border: solid thin gray;"><code><br />public class EmployeeBuilder {<br />    private Employee employee;<br /><br />    public EmployeeBuilder() {<br />        employee = new Employee();<br />    }<br /><br />    public static EmployeeBuilder defaultValues() {<br />        return new EmployeeBuilder();<br />    }<br /><br />    public static EmployeeBuilder clone(Employee toClone) {<br />        EmployeeBuilder builder = defaultValues();<br />        builder.setId(toClone.getId());<br />        builder.setName(toClone.getName());<br />        builder.setDepartment(toClone.getDepartment());<br />        return builder;<br />    }<br /><br />    public static EmployeeBuilder random() {<br />        EmployeeBuilder builder = defaultValues();<br />        builder.setId(getRandomInteger(0, 1000));<br />        builder.setName(getRandomString(20));<br />        builder.setDepartment(Department.values()[getRandomInteger(0, Department.values().length - 1)]);<br />        return builder;<br />    }<br /><br />    public EmployeeBuilder setId(int id) {<br />        employee.setId(id);<br />        return this;<br />    }<br /><br />    public EmployeeBuilder setName(String name) {<br />        employee.setName(name);<br />        return this;<br />    }<br /><br />    public EmployeeBuilder setDepartment(Department dept) {<br />        employee.setDepartment(dept);<br />        return this;<br />    }<br /><br />    public Employee build() {<br />        return employee;<br />    }<br />}<br /></code></pre><br />As you can see we have some factory methods: <br /><pre style="border: solid thin gray;"><code><br />    public static EmployeeBuilder defaultValues()<br />    public static EmployeeBuilder clone(Employee toClone)<br />    public static EmployeeBuilder random()<br /></code></pre><br />These methods return different builders:<br /><ul><li>defaultValues : some hardcoded values for each fields ( or the Java defaults - current implementation)</li><li>clone : will take all the values from the initial object, and give you the possibility to change just some</li><li>random : will generate random values for each field. This is very useful when you have a lot of fields that you don't specifically need in your test, but you need them to be initialized. getRandom* methods are defined statically in another class.</li></ul>&nbsp;You can add other methods that will initialized your builder accordingly to your needs.<br /><br />Also the builder can handle building some objects that are not so easily constructed and changed. For example let's change a little bit the Employee object and make it immutable:<br /><br /><br /><pre style="border: solid thin gray;"><code><br />public class Employee {<br />    private final int id;<br />    private final String name;<br />    private final Department department;<br />    ...<br />}<br /></code></pre><br />Now we lost the&nbsp;possibility&nbsp;to change the fields as we wish. But using the builder in the following form we can regain this possibility when constructing the object:<br /><br /><pre style="border: solid thin gray;"><code><br />public class ImmutableEmployeeBuilder {<br />    private int id;<br />    private String name;<br />    private Department department;<br /><br />    public ImmutableEmployeeBuilder() {<br />    }<br /><br />    public static ImmutableEmployeeBuilder defaultValues() {<br />        return new ImmutableEmployeeBuilder();<br />    }<br /><br />    public static ImmutableEmployeeBuilder clone(Employee toClone) {<br />        ImmutableEmployeeBuilder builder = defaultValues();<br />        builder.setId(toClone.getId());<br />        builder.setName(toClone.getName());<br />        builder.setDepartment(toClone.getDepartment());<br />        return builder;<br />    }<br /><br />    public static ImmutableEmployeeBuilder random() {<br />        ImmutableEmployeeBuilder builder = defaultValues();<br />        builder.setId(getRandomInteger(0, 1000));<br />        builder.setName(getRandomString(20));<br />        builder.setDepartment(Department.values()[getRandomInteger(0, Department.values().length - 1)]);<br />        return builder;<br />    }<br /><br />    public ImmutableEmployeeBuilder setId(int id) {<br />        this.id = id;<br />        return this;<br />    }<br /><br />    public ImmutableEmployeeBuilder setName(String name) {<br />        this.name = name;<br />        return this;<br />    }<br /><br />    public ImmutableEmployeeBuilder setDepartment(Department dept) {<br />        this.department = dept;<br />        return this;<br />    }<br /><br />    public ImmutableEmployee build() {<br />        return new ImmutableEmployee(id, name, department);<br />    }<br />}<br /></code></pre><br />This is very useful when we have hard to construct objects, or we need to change fields that are final.<br /><br />An here its the final result:<br /><br />Without builders:<br /><br /><pre style="border: solid thin gray;"><code><br />    @Test<br />    public void changeRoleTestWithoutBuilders() {<br />        // building the initial state<br />        Employee employee = new Employee();<br />        employee.setId(1);<br />        employee.setDepartment(Department.DEVELOPEMENT);<br />        employee.setName("John Johnny");<br /><br />        // testing the SUT<br />        EmployeeManager employeeManager = new EmployeeManager();<br />        employeeManager.changeRole(employee, Department.MANAGEMENT);<br /><br />        // building the expectations<br />        Employee expectedEmployee = new Employee();<br />        expectedEmployee.setId(employee.getId());<br />        expectedEmployee.setDepartment(Department.MANAGEMENT);<br />        expectedEmployee.setName(employee.getName());<br /><br />        // assertions<br />        assertThat(employee, is(expectedEmployee));<br />    }<br /></code></pre><br />With builders:<br /><br /><pre style="border: solid thin gray;"><code><br />    @Test<br />    public void changeRoleTestWithBuilders() {<br />        // building the initial state<br />        Employee employee = EmployeeBuilder.defaultValues().setId(1).setName("John Johnny").setDepartment(Department.DEVELOPEMENT).build();<br /><br />        // building the expectations<br />        Employee expectedEmployee = EmployeeBuilder.clone(employee).setDepartment(Department.MANAGEMENT).build();<br /><br />        // testing the SUT<br />        EmployeeManager employeeManager = new EmployeeManager();<br />        employeeManager.changeRole(employee, Department.MANAGEMENT);<br /><br />        // assertions<br />        assertThat(employee, is(expectedEmployee));<br />    }<br /></code></pre><br />As you can see, the size of the test is much smaller, and the construction of objects became much simpler (and nicer if you have a better code format). The difference is greater if you have a more complex domain object (which is more likely in real-life applications and especially in legacy code).<br /><br />Have fun!<br /><br /> <p><em>Meta: this post is part of the <a href="http://javaadvent.com/">Java Advent Calendar</a> and is licensed under the <a href="https://creativecommons.org/licenses/by/3.0/">Creative Commons 3.0 Attribution</a> license. If you like it, please spread the word by sharing, tweeting, FB, G+ and so on! Want to write for the blog? We are looking for contributors to fill all 24 slot and would love to have your contribution! <a href="mailto:dify.ltd@gmail.com">Contact Attila Balazs</a> to contribute!</em></p>