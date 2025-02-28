# `precice-config.xml`编写

1. `<log>`标签

   **日志记录配置**

   可能用到的占位符或变量

   | 占位符                | 描述                                                         |
   | --------------------- | ------------------------------------------------------------ |
   | ` %TimeStamp%`        | `%TimeStamp%`日志记录的时间戳<br />可以指定格式，例如 `%TimeStamp(format=“%Y-%m-%d %H：%M：%S”)%` |
   | `%Runtime%`           | 程序运行的时间                                               |
   | `%Severity%`          | 日志的严重程度级别，包括： <br />`trace`：最详细的日志信息，记录程序的每一处运行细节 <br />`debug`：用于调试的详细信息 <br />`info`：一般信息，用于记录程序的正常运行状态<br />`warning`：警告信息，程序运行中出现潜在问题，但尚未影响正常功能<br />`error`：错误信息，程序运行中出现严重问题，可能影响正常功能 <br />`critical`：极其严重的错误，会导致程序崩溃或无法继续运行 |
   | `%ColorizedSeverity%` | 带颜色的日志严重程度级别，便于在终端中区分                   |
   | `%Message%`           | 日志的具体消息内容                                           |
   | `%File%`              | 生成日志的文件名                                             |
   | `%Line%`              | 生成日志的代码行号                                           |
   | `%Function%`          | 生成日志的函数名                                             |
   | `%Module%`            | 生成日志的模块名                                             |
   | `%Rank%`              | 并行计算中的进程编号                                         |

   + `<sink>`标签：日志记录的输出通道

     | 属性      | 描述                           | 示例值                                              | default  |
     | --------- | ------------------------------ | --------------------------------------------------- | -------- |
     | `filter`  | 设置日志过滤条件               | `%Severity% > debug and %Rank% =0`<br />            | 无       |
     | `format`  | 设置日志格式                   | `---[precice] %Severity% %Message%`                 | 无       |
     | `enabled` | 设置是否启用该标签             | `true` 或 `false`                                   | `true`   |
     | `file`    | 设置日志文件的名称             | `precice.log`                                       | 无       |
     | `append`  | 设置是否将日志附加到现有文件中 | `true` ：附加<br />`false`：覆盖                    | `false`  |
     | `type`    | 设置日志的输出类型             | `"stream"` ：输出到流<br />`"file"`： 输出到文件    | `stream` |
     | `output`  | 设置日志输出的形式             | `"stdout"` ：标准输出<br />`"stderr"`：标准错误输出 | `stdout` |

   > 案例

   ```xml
   <log>
      <!-- 
       sink 配置说明：
       - 过滤条件：只记录日志级别高于 debug 的信息，并且如果日志级别为 info，则只记录进程编号为 0 的日志。
       - 日志格式：包含进程编号、时间戳、模块名、代码行号、函数名、带颜色的日志级别和消息内容。
   	- 输出形式：标准输出，日志输出到控制台。
   	- 输出类型：流
   	- 是否启用：启用
     -->
     <sink
       filter="(%Severity% > debug) and not ((%Severity% = info) and (%Rank% != 0))" 
       format="(%Rank%) %TimeStamp% [%Module%]:%Line% in %Function%: %ColorizedSeverity%%Message%" 
       output="stdout"
       type="stream"
       enabled="true" />
   </log>
   ```

2. `<profiling>`标签

   **性能分析工具**

   | 属性          | 描述                 | 示例值                                                       |
   | ------------- | -------------------- | ------------------------------------------------------------ |
   | `mode`        | 设置性能分析模式     | `fundamental`：基本模式，收集基本的性能数据。 <br />`advanced`：高级模式，收集更详细的性能数据。<br />`custom`：自定义模式，通过配置文件或代码自定义性能分析。 |
   | `synchronize` | 设置是否启动数据同步 | `true`：启用同步<br />`false` ：不启用同步                   |

   > 案例

   ```xml
   <!--
   	profiling配置说明：
   	- 性能分析模式：基本
   	- 是否启用时间同步：否的
   -->
   <profiling mode="fundamental" synchronize="false" />
   ```

3. `date:vector`标签

   **定义数据交换的矢量数据类型**

   | 属性   | 描述           | 默认值 | 是否必需 |
   | ------ | -------------- | ------ | -------- |
   | `name` | 数据类型的名称 | 无     | 是       |

   > 案例xml

   ```xml
   <data:vector name="Force" /> <!-- 定义力数据 -->
   <data:vector name="Displacement" /> <!-- 定义位移数据 -->
   ```

4. `<mesh>`标签

   **定义网格**

   | 属性         | 描述       | 类型   | 默认值 | 是否必需 |
   | ------------ | ---------- | ------ | ------ | -------- |
   | `name`       | 网格的名称 | string | 无     | 是       |
   | `dimensions` | 网格的维度 | int    | 无     | 是       |

   + `<use-data>`标签：设置网格所用的数据类型

     | 属性   | 描述               | 默认值 | 是否必需 |
     | ------ | ------------------ | ------ | -------- |
     | `name` | 所用数据类型的名称 | 无     | 是       |

   > 案例

   ```xml
   <mesh name="Fluid-Mesh" dimensions="2"> <!-- 网格名称为"Fluid-Mesh"，二维网格 -->
     <use-data name="Force" /> <!-- 该网格使用 Force 数据 -->
     <use-data name="Displacement" /> <!-- 该网格使用 Displacement 数据 -->
   </mesh>
   
   <mesh name="Solid-Mesh" dimensions="2"> <!-- 网格名称为"Solid-Mesh"，二维网格 -->
     <use-data name="Displacement" /> <!-- 该网格使用 Displacement 数据 -->
     <use-data name="Force" /> <!-- 该网格使用 Force 数据 -->
   </mesh>
   ```

5. `<participant>`标签

   **定义参与耦合的求解器的配置**

   | 属性   | 描述         | 默认值 | 是否必需 |
   | ------ | ------------ | ------ | -------- |
   | `name` | 求解器的名称 | 无     | 是       |

   + `<provide-mesh>`：设置求解器提供的网格名称

     | 属性   | 描述           | 默认值 | 是否必需 |
     | ------ | -------------- | ------ | -------- |
     | `name` | 提供的网格名称 | 无     | 是       |

   + `<receive-mesh>`：设置求解器接收的网格名称和来源

     | 属性   | 描述                       | 默认值 | 是否必需 |
     | ------ | -------------------------- | ------ | -------- |
     | `name` | 接收的网格名称             | 无     | 是       |
     | `from` | 网格的来源（提供者的名称） | 无     | 是       |

   + `<read-data>`：设置求解器从指定网格读取的数据名称

     | 属性   | 描述               | 默认值 | 是否必需 |
     | ------ | ------------------ | ------ | -------- |
     | `name` | 读取的数据名称     | 无     | 是       |
     | `mesh` | 读取数据的网格名称 | 无     | 是       |

   + `<write-data>`：设置求解器向指定网格写入的数据名称

     | 属性   | 描述               | 默认值 | 是否必需 |
     | ------ | ------------------ | ------ | -------- |
     | `name` | 写入的数据名称     | 无     | 是       |
     | `mesh` | 写入数据的网格名称 | 无     | 是       |

   + `<mapping:rbf>`：定义数据映射的配置

     | 属性         | 描述                  | 示例值                                                       | 是否必需 |
     | ------------ | --------------------- | ------------------------------------------------------------ | -------- |
     | `direction`  | 数据映射的方向        | `read`：读入数据的方向 <br />`write`：写出数据的方向         | 是       |
     | `from`       | 数据来源的网格名称    | 无                                                           | 是       |
     | `to`         | xml数据目标的网格名称 | 无                                                           | 是       |
     | `constraint` | 数据映射的约束条件    | `consistent`：一致性。确保映射后的数据在目标网格上保持一致，即数据的值在目标网格上是准确的。<br />`conservative`：保守性。确保映射后的数据在目标网格上保持守恒，即数据的总量在映射前后保持不变。 | 是       |

     + `<basis-function>`标签

       **设置数据映射的基函数类型，基函数用于数据在不同网格之间的平滑过渡**

       | 基函数类型                                | 数学表达式                                                  | 特点                                       | 应用场景                                             |
       | ----------------------------------------- | ----------------------------------------------------------- | ------------------------------------------ | ---------------------------------------------------- |
       | **线性基函数 (Linear)**                   | $\phi(x) = x$                                               | 简单、计算效率高                           | 适用于线性问题                                       |
       | **多项式基函数 (Polynomial)**             | $\phi(x) = x^n$，其中 $n$ 是多项式的阶数                    | 可以捕捉非线性关系，但可能导致过拟合       | 适用于非线性问题，但需要谨慎选择多项式的阶数         |
       | **高斯基函数 (Gaussian)**                 | $\phi(x) = \exp\left(-\frac{(x - \mu)^2}{2\sigma^2}\right)$ | 具有无限光滑性，受形状参数 $\sigma$ 影响   | 适用于需要平滑插值的问题，如动态和不规则域的数据处理 |
       | **多二次基函数 (Multiquadric)**           | $\phi(x) = \sqrt{1 + (\epsilon x)^2}$                       | 具有无限光滑性，受形状参数 $\epsilon$影响  | 适用于高维问题的插值和逼近                           |
       | **逆多二次基函数 (Inverse Multiquadric)** | $\phi(x) = \frac{1}{1 + (\epsilon x)^2}$                    | 具有无限光滑性，受形状参数 $\epsilon$ 影响 | 适用于高维问题的插值和逼近                           |
       | **逆二次基函数 (Inverse Quadric)**        | $\phi(x) = \frac{1}{\sqrt{1 + (\epsilon x)^2}}$             | 具有无限光滑性，受形状参数 $\epsilon$影响  | 适用于高维问题的插值和逼近                           |
       | **薄板样条基函数 (Thin-Plate Spline)**    | $\phi(x) = x^2 \ln(x)$                                      | 分段光滑，无形状参数                       | 适用于二维和三维数据的平滑插值                       |
       | **线性径向基函数 (Linear Radial)**        | $\phi(x) = x$                                               | 简单、计算效率高                           | 适用于简单的非线性问题                               |
       | **三次基函数 (Cubic)**                    | $\phi(x) = x^3$                                             | 可以捕捉非线性关系                         | 适用于需要平滑插值的问题                             |
       | **单项式基函数 (Monomial)**               | $\phi(x) = x^{2k-1}$，其中 $k$是自然数                      | 可以捕捉非线性关系                         | 适用于需要高阶多项式插值的问题                       |

   + `<watch-point>`：设置网格上设置监视点

     | 属性         | 描述                 | 默认值 | 是否必需 |
     | ------------ | -------------------- | ------ | -------- |
     | `mesh`       | 监视点所在的网格名称 | 无     | 是       |
     | `name`       | 监视点的名称         | 无     | 是       |
     | `coordinate` | 监视点的坐标         | 无     | 是       |

   + `<export>`：设置数据导出的配置

     | 属性        | 描述           | 默认值 | 是否必需 |
     | ----------- | -------------- | ------ | -------- |
     | `directory` | 导出数据的目录 | 无     | 是       |

   > 案例

   ```xml
   <!-- 定义Fluid求解器的配置 -->
   <participant name="Fluid"> <!-- 求解器的名称为 "Fluid" -->
     <provide-mesh name="Fluid-Mesh" /> <!-- Fluid求解器提供的网格名称为"Fluid-Mesh" -->
     <receive-mesh name="Solid-Mesh" from="Solid" /> <!-- 接收来自"Solid"求解器的网格 "Solid-Mesh" -->
     <read-data name="Displacement" mesh="Fluid-Mesh" /> <!-- 从"Fluid-Mesh"网格读取"Displacement" 数据 -->
     <write-data name="Force" mesh="Fluid-Mesh" /> <!-- 向"Fluid-Mesh"网格写入"Force"数据 -->
     <!-- 数据映射配置 -->
     <mapping:rbf direction="read" from="Solid-Mesh" to="Fluid-Mesh" constraint="consistent"> <!-- 数据读取映射，从 "Solid-Mesh" 映射到 "Fluid-Mesh"，约束条件为 "consistent" -->
       <basis-function:thin-plate-splines /> <!-- 使用薄板样条函数作为基函数 -->
     </mapping:rbf>
     <mapping:rbf direction="write" from="Fluid-Mesh" to="Solid-Mesh" constraint="conservative"> <!-- 数据写入映射，从 "Fluid-Mesh" 映射到 "Solid-Mesh"，约束条件为 "conservative" -->
       <basis-function:thin-plate-splines /> <!-- 使用薄板样条函数作为基函数 -->
     </mapping:rbf>
   </participant>
   
   <!-- 定义Solid求解器的配置 -->
   <participant name="Solid"> <!-- 参与者的名称为"Solid" -->
     <provide-mesh name="Solid-Mesh" /> <!-- 提供的网格名称为"Solid-Mesh" -->
     <read-data name="Force" mesh="Solid-Mesh" /> <!-- 从"Solid-Mesh"网格读取"Force" 数据 -->
     <write-data name="Displacement" mesh="Solid-Mesh" /> <!-- 向"Solid-Mesh"网格写入"Displacement"数据 -->
     <!-- 监视点和 VT 文档输出 -->
     <watch-point mesh="Solid-Mesh" name="Flap-Tip" coordinate="0.25;0.0" /> <!-- 在"Solid-Mesh"网格上设置监视点"Flap-Tip"，坐标为 (0.25, 0.0) -->
     <export:vtk directory="precice-exports" /> <!-- 将数据以VTK格式导出到"precice-exports"目录 -->
   </participant>
   ```

6. `<m2n:sockets>`标签

7. `<coupling-scheme:serial-explicit>`标签
