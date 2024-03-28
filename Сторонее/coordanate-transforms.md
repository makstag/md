# Преобразования из одной системы отсчета отсчета в другую
# Понятие матрицы перехода от старого базиса к новому
Пусть $(e^{'}_1, e^{'}_2)$ - новый базис, $(e_1, e_2)$ - старый базис. Разложение базисных векторов нового базиса в старом базисе запишется в виде:
$$ e^{'}_{1} = a_{11} e_{1} + a_{21} e_{2} $$ $$ e^{'}_{2} = a_{21} e_{1} + a_{22} e_{2}.$$
Сформируем матрицу, в которой коэффициенты разложения по страрому базису запишем поочередно в столбцы, получим:
$$ A = \pmatrix{a_{11} & a_{12}\\ a_{21} & a_{22}}.$$
Такую матрицу принято называть **матрицей перехода от старого базиса к новому базису**.
Пусть $V =(v_1,v_2)$ координаты некоторой точки в старом базисе,  $V^{'} =(v^{'}_1,v^{'}_2)$ - координаты этой же точки в новом базисе. Тогда координаты точки в новом базисе, можно получить зная координаты точки в старом базисе следующим образом:
$$V^{'} = A^{-1} V,$$
где $A^{-1}$ - обратная матрица к матрице перехода $A$.
Аналогичное справедливо для векторов любых размерностей.
# Понятие позы датчика на примере камеры

Для проверки качества работы методов SLAM используется понятие позы датчика (левой камеры).

Поза $Pose$ в этом случае представляет собой **матрицу преобразования** из системы координат $x$,$y$, $z$ в систему координат $x_{0},\ y_{0},\ z_{0}$.
Она имеет размер 3x4 или в общем 4x4, чтобы мы имели возможность осуществлять над ней операцию обращения.

Каждый элемент матрицы $Pose$ имеет физический смысл (см. рисунок). Проще рассматривать столбцы этой матрицы.

![[tf.png]]

Последний столбец матрицы $Pose$ - это проекции начала системы координат датчика $t_{x0},\ t_{y0},\ t_{z0}$ на оси $x_{0},\ y_{0},z_{0}$ 0-й системы координат}, соответствующей 0-й позе датчика.

В первом столбце матрицы $Pose$ $x_{x0},\ x_{y0},x_{z0}$ - проекции вектора $x$ системы координат датчика на оси $x_{0},\ y_{0},z_{0}$ 0-й системы координат, соответствующей 0-й позе датчика. Эти проекции представляют собой значения косинусов углов соответственно между осью $x$ и $x_{0}$, осью $x$ и $y_{0}$, осью $x$ и $z_{0}$(т.к. вектора $x$, $y$, $z$ и  $x_{0},\ y_{0},z_{0}$ - орты).

Во втором столбце матрицы $Pose$ $y_{x0},\ y_{y0},\ y_{z0}$ - проекции вектора $y$ системы координат датчика на оси $x_{0},\ y_{0},z_{0}$ 0-й системы координат}, соответствующей 0-й позе датчика. Эти проекции представляют собой значения косинусов углов соответственно между осью $y$ и $x_{0}$, осью $y$ и $y_{0}$, осью $y$ и $z_{0}$.

В третьем столбце матрицы $Pose$  $z_{x0},\ z_{y0},\ z_{z0}$ - проекции вектора $z$ системы  координат датчика на оси $x_{0},\ y_{0},z_{0}$ 0-й системы координат, соответствующей 0-й позе датчика. Эти проекции представляют собой значения косинусов углов соответственно между осью $z$ и $x_{0}$, осью $z$ и $y_{0}$, осью $z$ и $z_{0}$.

Для такой интерпретации 0-я поза датчика должна представлять собой единичную матрицу. Для краткости будем обозначать $Pose$ через P.

$$P = \pmatrix{1 & 0 & 0 & 0\\
			   0 & 1 & 0 & 0\\
			   0 & 0 & 1 & 0\\
			   0 & 0 & 0 & 1}$$
Если 0-я поза не имеет такой вид (из-за того что посчитана относительно какой-то другой системы координат), то ее можно легко привести к единичной по формуле
$$P_{0} = P_{0}^{- 1} \times P_{0},$$
где x - операция матричного умножения, -1 - операция обращения матрицы.
При этом все остальные i-е позы из последовательности нужно
пересчитать аналогично:
$$P_{i} = P_{0}^{- 1} \times P_{i}$$

# Переход между системами координат разных датчиков  
Пусть в качестве ground truth (истинных) поз выступают позы датчика Atlans, который объединяет данные GPS/ГЛОНАСС с RTK-поправками, колесного одометра:

$$P_{0}^{A},P_{1}^{A},...\ ,P_{i}^{A},\ ...\ ,P_{n}^{A},$$
где $n$ - количество поз.

Пусть нужно на основе имеющихся поз датчика Atlans получить позы левой камеры (Camera):
$$P_{0}^{С},P_{1}^{С},...\ ,P_{i}^{С},\ ...\ ,P_{n}^{С},$$
Для этого должна иметься матрица $T_{C2A}$ преобразования от системы координат CameraL к системе координат Atlans, полученная в результате процедуры калибровки.

![Пояснения к выводу матрицы позы одного датчика через матрицу](attachments/4279a0307637ffbc1a7110fe45ddfc7c.png)
Рассмотрим точку \emph{X}, ее координаты $X{}_{0}^{A}$ в системе координат 0-й позы и $X{}_{i}^{A}$ в системе координат i-й позы датчика Atlans связаны друг с другом как
$$X_{0}^{A} = P_{i}^{A}X_{i}^{A}$$

  

ее координаты $X_{0}^{С}$ в системе координат 0-й позы $X_{i}^{C}$ в системе координат i-й позы левой камеры (CameraL) связаны друг с другом как  
$$X_{0}^{C} = P_{i}^{C}X_{i}^{C}.$$
На основе матрицы преобразования $T_{C2A}$ получаем
$$X_{0}^{A} = T_{C2A}X_{0}^{C}$$
$$X{i}^{A} = T_{C2A}X_{i}^{C}$$
Далее приравниваем $X_{0}^{A}$: 
$$X_{0}^{A} = P_{i}^{A}X_{i}^{A} = T_{C2A}X_{0}^{C}$$
И подставляем сюда $X_{i}^{A}$
  $$P_{i}^{A}T_{C2A}X_{i}^{C} = T_{C2A}X_{0}^{C}$$
Получаем
$$X_{0}^{C} = T_{C2A}^{- 1}P_{i}^{A}T_{C2A}X_{i}^{C} = P_{i}^{C}X_{i}^{C}$$
Отсюда искомая матрица позы левой камеры
$$P_{i}^{C} = T_{C2A}^{- 1}P_{i}^{A}T_{C2A}.$$
Если имеется матрица преобразования $T_{L2C}$ из системы координат Lidar в CameraL и матрица преобразования $T_{L2A}$ из системы координат Lidar в Atlans, то матрицу преобразования $T_{C2A}$ из системы координат CameraL в Atlans можно вычислить
$$T_{C2A} =T_{L2C}^{- 1}T_{L2A}.$$
И матрица позы левой камеры примет вид
$$P_{i}^{C} = T_{L2A}^{- 1}T_{L2C}P_{i}^{A}T_{L2C}^{- 1}T_{L2A}.$$
Аналогично искомая матрица позы лидара 
$$P_{i}^{L} = T_{L2A}^{- 1}P_{i}^{A}T_{L2A}.$$

# Преобразование позы (трансляция + ориентация)

Пусть $P$ - искомая новая поза (матрица 4х4), $P_0$ - исходная старая поза (матрица 4х4), $Т$ - матрица преобразования (4х4) из старой системы координат в новую. Матрица 4x4 имеет вид $\pmatrix{R & t \\ 0 & 1}$, где $R$ - матрица поворота, $t$ - вектор трансляции.
Для преобразования нужно домножить на матрицу преобразования слева:
$$P = T P_0. $$
Если $q$ - кватернион соответствующий матрице $R$. Тогда преобразование с использованием кватерниона $q$ и вектора трансляции $t$ записывается следующим образом:
$$t_{P_{0}} =  q_T * t_P + t_T,$$
$$q_{P_0} =  q_{P} * q_T ,$$
где $q_T*t_P$ поворот вектора $t_P$ на значение задаваемое кватернионом $q_T$.

## Примеры кода
_На С++_  
[https://answers.ros.org/question/261419/tf2-transformpose-in-c/](https://answers.ros.org/question/261419/tf2-transformpose-in-c/)
```
#include <tf2_geometry_msgs/tf2_geometry_msgs.h>
#include <tf2_ros/transform_listener.h>

tf2_ros::Buffer tf_buffer;
tf2_ros::TransformListener tf2_listener(tf_buffer);

geometry_msgs::TransformStamped base_link_to_leap_motion; // My frames are named "base_link" and "leap_motion"

base_link_to_leap_motion = tf_buffer.lookupTransform("leap_motion","base_link", ros::Time(0), ros::Duration(1.0));

tf2::doTransform(robot_pose, robot_pose, base_link_to_leap_motion); // robot_pose is the PoseStamped I want to transform
```
  

_На PYTHON_
[https://answers.ros.org/question/323075/transform-the-coordinate-frame-of-a-pose-from-one-fixed-frame-to-another/](https://answers.ros.org/question/323075/transform-the-coordinate-frame-of-a-pose-from-one-fixed-frame-to-another/)
```
# Transform a given input pose from one fixed frame to another
import rospy
from geometry_msgs.msg import Pose
import tf2_ros
import tf2_geometry_msgs  # **Do not use geometry_msgs. Use this instead for PoseStamped

def transform_pose(input_pose, from_frame, to_frame):
    # **Assuming /tf2 topic is being broadcasted
    tf_buffer = tf2_ros.Buffer()
    listener = tf2_ros.TransformListener(tf_buffer)
    pose_stamped = tf2_geometry_msgs.PoseStamped()
    pose_stamped.pose = input_pose
    pose_stamped.header.frame_id = from_frame
    pose_stamped.header.stamp = rospy.Time.now()
    try:
        # ** It is important to wait for the listener to start listening. Hence the rospy.Duration(1)
        output_pose_stamped = tf_buffer.transform(pose_stamped, to_frame, rospy.Duration(1))
        return output_pose_stamped.pose
    except (tf2_ros.LookupException, tf2_ros.ConnectivityException, tf2_ros.ExtrapolationException):
        raise
# Test Case
rospy.init_node("transform_test")

my_pose = Pose()
my_pose.position.x = -0.25
my_pose.position.y = -0.50
my_pose.position.z = +1.50
my_pose.orientation.x = 0.634277921154
my_pose.orientation.y = 0.597354098852
my_pose.orientation.z = 0.333048372508
my_pose.orientation.w = 0.360469667089
transformed_pose = transform_pose(my_pose, "fixture", "world")
print (transformed_pose)
```  

# Преобразование ковариации
## Формула от Ильи Белкина
$$ cov (x^*) = C_{J_{x}}cov(y)C_{J_{x}}^{T} = \frac{dx}{dz} C_{J_{z}} cov(y) C_{J_{z}}^{T} (\frac{dx}{dz})^{T} = \frac{dx}{dz} cov(z^{*}) \frac{dz}{dx}.$$
## С помощью MRPT
Преобразование ковариации из одной системы координат в другую на С++ выполняется с помощью библиотеки [MRPT](http://www.mrpt.org/) с помощью метода [changeCoordinatesReference](http://mrpt.ual.es/reference/stable/class_mrpt_poses_CPose3DPDFGaussian.html?highlight=cpose3dpdfgaussian#doxid-classmrpt-1-1poses-1-1-c-pose3-d-p-d-f-gaussian-1a3347d655f2ff1bbb232a6b3bb4ed77e8) класса [CPose3DPDFGaussian](https://github.com/MRPT/mrpt/blob/develop/libs/poses/src/CPose3DPDFGaussian.cpp).

Прямые вычисления для ковариации курса, крена, тангажа слишком сложны. Вместо этого делает [обходной способ](https://github.com/MRPT/mrpt/blob/develop/libs/poses/src/CPose3DPDF.cpp#L137-L158):

      `X(6D)           U(6D)`
       `|                |`
       `v                v`
     `X(7D)            U(7D)`
       `|                |`
       `+------ (+) -----+`
                `|`
                `v`
             `RES(7D)`
                `|`
                `v`
             `RES(6D)`

```
FUNCTION = f_quat2eul( f_quat_comp( f_eul2quat(x) , f_eul2quat(u) ) )

Jacobians chain rule:

JACOB_dx = J_Q2E (6x7) * quat_df_dx (7x7) * J_E2Q_dx (7x6)

JACOB_du = J_Q2E (6x7) * quat_df_du (7x7) * J_E2Q_du (7x6)  
```

Переменные J_E2Q_dx, J_E2Q_du размерностью 7х6 вначале инициализируются нулями. Далее, элементы по диагонали (0,0), (1,1), (2,2) приравниваются единице. Создается промежуточная матрица 4х3 (dq_dr_sub), которая представляет собой якобиан кватерниона по углам Эйлера

_С помощью функции_ [_transformCovariance_](http://docs.ros.org/en/melodic/api/tf2_geometry_msgs/html/c++/namespacetf2.html#a9388e662eaab80021cad30a2a75e4c63)

# Преобразование скорости

[http://wiki.ros.org/tf/Tutorials/Writing%20a%20tf%20listener%20%28C%2B%2B%29](http://wiki.ros.org/tf/Tutorials/Writing%20a%20tf%20listener%20%28C%2B%2B%29)

```
geometry_msgs::Twist vel_msg;
vel_msg.angular.z = 4.0 * atan2(transform.getOrigin().y(),
                                transform.getOrigin().x());
vel_msg.linear.x = 0.5 * sqrt(pow(transform.getOrigin().x(), 2) +
```

Здесь преобразование используется для вычисления новых линейных и угловых скоростей для turtle2 на основе расстояния и угла от turtle1. Новая скорость опубликована в топике "turtle2/cmd_vel", и SIM будет использовать ее для обновления движения turtle2.

Возможный ответ на вопрос - [https://answers.ros.org/question/192273/how-to-implement-velocity-transformation/](https://answers.ros.org/question/192273/how-to-implement-velocity-transformation/)

В этом возможном ответе говорят, что скорость можно представить в виде вектора, и тогда трансформацию можно выполнить с помощью трансформации вектора [transformVector](http://docs.ros.org/en/hydro/api/tf/html/c++/classtf_1_1TransformListener.html#a6c3cd856bf584f98db6f0f295c2d9f52). Но другие говорят, что есть неопределенность в целевой системе координат.


[Здесь](http://docs.ros.org/en/hydro/api/tf/html/c++/transform__listener_8cpp_source.html#L00111) есть функция transformTwist, но она закомменчена ввиду неоднозначности в представлении сообщения в ROS. Неоднозначность Эта неоднозначность описана [здесь](http://wiki.ros.org/tf/Reviews/2010-03-12_API_Review). 

Проблема со скоростями в сообщениях состоит в том, что с любой скоростью связаны 4 потенциально различных систем координат:
-   reference point
-   reference frame
-   observational frame
-   body frame

Но в сообщениях типа [TwistStamped](http://wiki.ros.org/TwistStamped) присутствует только одна система координат.

Пример неоднозначности: два робота движутся прямо. Каждый видит, что другой робот движется со скоростью 1 м/с в мировой системе координат. Допустим мы захотели узнать скорость робота 1 относительно робота 2. Результатом этого может быть преобразование линейной скорости 1 м / с в комбинацию углового и линейного, которая компенсируется расстоянием между роботами. 

[Здесь](http://docs.ros.org/en/hydro/api/tf/html/c++/tf_8cpp_source.html#l00466) есть метод [lookupTwist](http://docs.ros.org/en/hydro/api/tf/html/c++/tf_8cpp_source.html), который ищет трансформацию по скорости по вышеприведенным 4-м системам координат (tracking_frame, observation_frame, reference_frame, reference_point_frame) и по reference_point. В этом методе скорость определяется усреднением за некоторый промежуток averaging_interval. Находятся трансформации из tracking_frame в observation_frame  в начальный момент времени (назовем его start) и в некоторый промежуток времени (averaging_interval / 2) спустя (назовем этот момент времени- end). Определяется перемещение по трансляции между этими двумя моментами времени, а затем и тангенциальная скорость при делении полученной трансляции на промежуток времени. Для определения угловой скорости вначале определяется относительное изменение ориентации между двумя промежутками времени (start, end) в виде кватерниона. Кватернион относительного изменения ориентации переводится в трехмерный вектор (yaw, pitch, roll), считается скорость изменения yaw, pitch, roll за заданный промежуток времени (averaging_interval / 2). Полученная скорость изменения Эйлеровых углов домножается слева на трехмерный вектор (yaw, pitch, roll) ориентации в конечный момент времени. 

Если скорость имеет только тангенциальные составляющие, то преобразование можно выполнить с помощью следующего [кода](https://answers.ros.org/question/192273/how-to-implement-velocity-transformation/):

```
# rotate vector v1 by quaternion q1 
 
def qv_mult(q1, v1):
 
    #v1 = t.unit_vector(v1)
    q2 = list(v1)
    q2.append(0.0)
    return t.quaternion_multiply(
        t.quaternion_multiply(q1, q2), 
        t.quaternion_conjugate(q1)
    )[:3]
 
#twist_rot is the velocity in euler angles (rate of roll, rate of pitch, rate of yaw; angular.x, angular.y, angular.z)
#twist_vel is the velocity in cartesian coordinates (linear.x, linear.y, linear.z)
 
def transform_twist(twist_rot, twist_vel):
    map_to_base_trans, map_to_base_rot = listener.lookupTransform("map", "base_footprint", rospy.Time());
    q1 = map_to_base_rot
    out_vel = qv_mult(q1, twist_vel)
    out_rot = twist_rot
    return out_vel, out_rot
 
#out_vel is the linear velocity w.r.t to the base_footpring, linear.x, linear.y, linear.z
#out_Rot is the angular velocity w.r.t. to the base_footprint, angular.x, angular.y, angular.z
out_vel, out_rot = transform_twist(twist_rot, twist_vel)
```

  
Если задана скорость объекта в системе отсчета В, и нужно найти скорость объекта в системы отсчета А, и если известна матрица 4х4 преобразования из системы отсчета А в В, то можно вывести скорость объекта в системе отсчета А. Скорость одного и того же объекта в разных системах координат связаны сопряженной картой.

Пусь нам известна матрица преобразования из фрейма А в фрейм В:
$$ T = \left[\pmatrix{R & p \\ 0_{1\times3} & 1}\right]$$
Воспользуемся следующими соотношениями:
$$ w_{a} = R w_{b}$$
$$r_{a} = Rr_{b} + p$$
где
$w_{a}$- угловая скорость объекта в системе отсчета A,

$w_{b}$- угловая скорость объекта в системе отсчета A,

$r_{a}$- вектор, соединяющий ось вращения и объект в системе отсчета А.

$r_{b}$- вектор, соединяющий ось вращения и объект в системе отсчета B.

Линейная скорость объекта в системе отсчета А будет равна:
$$ v_{a} = -w_{a} \times r_{a}$$
# Преобразование скорости

[http://wiki.ros.org/imu_transformer](http://wiki.ros.org/imu_transformer)  

# Источники

1 Foote T. tf: The transform library //2013 IEEE Conference on Technologies for Practical Robot Applications (TePRA). – IEEE, 2013. – С. 1-6.

2 [Zhao, Shiyu. "Time derivative of rotation matrices: A tutorial." _arXiv preprint arXiv:1609.06088_ (2016).](https://arxiv.org/pdf/1609.06088.pdf)