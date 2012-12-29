#Booth乘法器报告

姓名：吴则有
学号：12212020030

##一、设计要求
完成16*16有符号乘法器的设计、验证工作。











###Booth乘法器

Booth算法是一种十分有效的计算有符号数乘法的算法。算法的新型之处在于减法也可用于计算乘积。Booth发现加法和减法可以得到同样的结果。因为在当时移位比加法快得多，所以Booth发现了这个算法，Booth算法的关键在于把1分类为开始、中间、结束三种，如下图所示

![principle4](./readme_pic/principle4.png)

当然一串0或者1的时候不操作，所以Booth算法可以归类为以下四种情况：

![principle5](./readme_pic/principle5.png)

Booth算法根据乘数的相邻2位来决定操作，第一步根据相邻2位的4中情况来进行加或减操作，第二部仍然是将积寄存器右移，算法描述如下：

####1.根据当前位和其右边的位，做如下操作：

*    00:	0的中间，无任何操作；
















端口 | 说明 
---------- | -------------
M [15:0]	|	输入端口，被乘数



采用的规则如下：

![design2](./readme_pic/design2.png)

经典Booth编码在编码方面相比较Booth2等编码，方法更简单，硬件实现更容易，但是带来的问题也是十分明显的。首先，经典Booth编码并没有减少部分积的数目，这就给之后Wallace树的构建制造了麻烦，这点我们会在之后的章节中提到；此外，因为有可能需要生成“-X”部分积，因此16位的带符号操作数还需要特别对符号位进行处理。




![principle9](./readme_pic/principle9.png)






* 每一层所有加法器的输出就被连接到2组线之中，比如Fir1_S[15:0]，Fir1_C[15:0]，Sec2_S[17:0]，Sec2_C[17:0]，以此类推。每组线的宽度有各层的加法器数量决定。


![wallace3](./readme_pic/wallace3.png)

![wallace4](./readme_pic/wallace4.png)

![wallace5](./readme_pic/wallace5.png)

![wallace6](./readme_pic/wallace6.png)

![wallace7](./readme_pic/wallace7.png)

Wallace树模块最后的输出结果就是两个32位的操作数opa/opb。将这两个操作数送入平方根进位选择加法器CS_Adder32后，就可以得到这16个部分积相加的结果。

###平方根进位选择加法器
除了上述两个模块外，整个乘法器还有一个不能被忽视的模块，那就是平方根进位选择加法器。












	input signed[15:0] a, b;
	output signed[31:0] p;
	assign p = a*b;

	endmodule
之后testbench的编写中，我们将生成200对符合范围内的16位随机有符号数，分别输入到乘法器和之前的check模块中，比对结果是否相符。如果有错误的结果，计数器就将加1。并保留当前计算结果以及之前的2个计算结果，方便出错时观察。

###testbench代码

以下是乘法器总的testbench的代码：

	`timescale 1ns/1ps
	
	module multiplier_tb;
	
	parameter	TCLK = 10;
	reg		clk;
	
	reg		[15: 0]	x, y;
	wire	[31: 0] res;
	wire   	[31: 0] res_check;
	
	initial	clk = 1'b0;
	always #(TCLK/2)	clk = ~clk;
	
	reg[31: 0] res_check1, res_check2, res_check3;
	reg[5 : 0] counter;
	initial counter = 0;
	always @(posedge clk)
	begin
   		res_check1 <= res_check;
    	res_check2 <= res_check1;
    	res_check3 <= res_check2;
    	if (res != res_check)
       		counter <= counter+1;
	end
	
	initial
	begin
    	repeat(200)
    	begin
       		x = {$random}%17'h10000;
       		y = {$random}%17'h10000;
       		#TCLK ;
    	end
    	$stop;
	end
	
	TopMultiplier	multiplier_test	(
										.x_in (x),
										.y_in (y),
										.result_out (res)
									);
	
	multiplier_check	multiplier_check0 (
											.a(x),
											.b(y),
											.p(res_check)
										);
	
	endmodule



![sim1](./readme_pic/wave1.jpg)

![sim2](./readme_pic/wave2.jpg)

可以看到计数器counter始终为零。说明两个模块在200个随机数的验证情况下，也没有出现不一致的情况。因此我们认为，这个乘法器模块的功能实现是正确的。

##五、逻辑综合

可以看到计数器counter始终为零。说明两个模块在200个随机数的验证情况下，也没有出现不一致的情况。因此我们认为，这个乘法器模块的功能实现是正确的。

###相关脚本文件
一共有2个相关的脚本文件：TopMultiplier_CONST.con和TopMultiplier.tcl。其中, TopMultiplier_CONST.con文件中定义了一些关键性的设计约束，而TopMultiplier.tcl文件则是顶层的逻辑综合流程,在综合时只需要执行top.tcl文件,就会调用其他的脚本文件。
###设计约束
####·时钟及输入输出延时
由于这个ALU是纯组合逻辑,在DC综合时需要加上一个虚拟时钟。时钟部分的设计约束如下:

![clock](./readme_pic/clock.png)

####·驱动和负载

![drive&load](./readme_pic/constraint_load.png)

###逻辑综合结果
####·性能
根据timing report，该电路的时钟频率约为305MHz。关键路径如下：

![critica_path](./readme_pic/critical_path.png)

####·功耗
根据DC report中的power部分可知,整个电路的动态功耗为32.1228mW,静态功耗为3.6027uw。由下图可以看出各部分所占功耗的百分比,因为全部由组合逻辑组成,组合逻辑部分占到了100%。

![power](./readme_pic/power.png)

####·面积
根据DC report中的power部分可知,整个电路的动态功耗为32.1228mW,静态功耗为3.6027uw。由下图可以看出各部分所占功耗的百分比,因为全部由组合逻辑组成,组合逻辑部分占到了100%。

![area](./readme_pic/area.png)

可以看到整个电路的面积还是非常大的。一方面是因为Wallace树中我仅仅采用了半加器和全加器，而没有使用4-2压缩器等模块；另一方面也由于我使用了三次CSAdder模块，在后续的设计中应该改进策略，把符号位的处理放到Wallace树模块中，应该可以进一步减小电路面积。