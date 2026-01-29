# Activity 01 - ROS2 Topics

Here is a representation of the graph you should get at the end. 

<img src="../recursos/imgs/diag.jpeg" alt="Diagrama del sistema" width="420">

Blue boxes are nodes, green are topics.

And here’s the final result you should get in the terminal (of course the numbers may be different): 

 

ros2 topic echo /number_count 

data: 38 

data: 40 

data: 42 

You’ll create 2 nodes from scratch. In the first one you’ll have 1 publisher, and in the second one, 1 publisher & 1 subscriber. 

The number_publisher node publishes a number (always the same) on the “/number” topic, with the existing type example_interfaces/msg/Int64. 
The number_counter node subscribes to the “/number” topic. It keeps a counter variable. Every time a new number is received, it’s added to the counter. The node also has a publisher on the “/number_count” topic. When the counter is updated, the publisher directly publishes the new value on the topic. 
A few hints: 

Check what to put into the example_interfaces/msg/Int64 with the “ros2 interface show” command line tool. 
It may be easier to do the activity in this order: first create the number_publisher node, check that the publisher is working with “ros2 topic”. Then create the number_counter, focus on the subscriber. And finally create the last publisher. 
In the number_counter node, the publisher will publish messages directly from the subscriber callback. 




---
##Diagram
<img src="../recursos/imgs/ibero.jpeg" alt="Diagrama del sistema" width="420">

## Codes

1. **Subscriber and publisher code**  

``` codigo

#!/usr/bin/env python3
#!/usr/bin/env python3

import rclpy
from rclpy.node import Node
from example_interfaces.msg import Int64


class NumberCounter(Node):

    def __init__(self):
        super().__init__('number_counter')

        self.counter = 0

        self.subscription = self.create_subscription(
            Int64,
            '/number',
            self.listener_callback,
            10
        )

        self.publisher_ = self.create_publisher(
            Int64,
            '/number_count',
            10
        )

        self.get_logger().info('Number Counter iniciado')

    def listener_callback(self, msg):
        self.counter += msg.data

        out_msg = Int64()
        out_msg.data = self.counter
        self.publisher_.publish(out_msg)

        self.get_logger().info(
            f'Recibido: {msg.data} | Total acumulado: {self.counter}'
        )


def main(args=None):
    rclpy.init(args=args)
    node = NumberCounter()
    rclpy.spin(node)
    rclpy.shutdown()


if __name__ == '__main__':
    main()

```
2. **Publisher code**  

``` codigo

#!/usr/bin/env python3

import rclpy
from rclpy.node import Node
from example_interfaces.msg import Int64


class NumberPublisher(Node):

    def __init__(self):
        super().__init__('number_publisher')

        self.publisher_ = self.create_publisher(
            Int64,
            '/number',
            10
        )

        self.timer = self.create_timer(1.0, self.timer_callback)

        self.number = 2  
        self.get_logger().info('Number Publisher iniciado')

    def timer_callback(self):
        msg = Int64()
        msg.data = self.number
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publicando: {msg.data}')


def main(args=None):
    rclpy.init(args=args)
    node = NumberPublisher()
    rclpy.spin(node)
    rclpy.shutdown()


if __name__ == '__main__':
    main()

```

2. **Setup py code**  

``` codigo

from setuptools import find_packages, setup

package_name = 'my_rob'

setup(
    name=package_name,
    version='0.0.0',
    packages=find_packages(exclude=['test']),
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='valeria',
    maintainer_email='valeria@todo.todo',
    description='TODO: Package description',
    license='TODO: License declaration',
    extras_require={
        'test': [
            'pytest',
        ],
    },
    entry_points={
        'console_scripts': [
            'r2d2 = my_rob.my_first_node:main',
            'number_publisher = myrob_pkg.number_publisher:main',
            'number_counter = my_rob.number_counter:main',
            'other_file = myrob_pkg.other_file:main',
            'number_publisher2 = my_rob.number_publisher2:main',
            'number_counter2 = my_rob.number_counter2:main',
        ],
    },
)

```