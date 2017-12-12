# Injecting Default Routes
1. Using `default-information originate`
2. Using Stub Areas

## default-information originate
The OSPF subcommand default-information originate tells OSPF to create a Type 5 LSA
(used for external routes) for a default route—0.0.0.0/0—and flood it like any other Type 5 LSA.
In other words, it tells the router to create and flood information about a default
route throughout the OSPF domain.