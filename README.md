# XIDIAN-micro-control-system
西安电子科技大学-通信工程学院-微控制系统项目设计-智能药箱

项目为智能药箱，项目目标:对药品进行分类

该项目由六人分组完成，职位有经理、秘书、公关、网页设计师、软件工程师、电子和布线工程师、组装工程师、机械设计工程师、数字系统工程师。


FPGA代码
1                                   
  ![image](https://github.com/COFFEEOfficial/XIDIAN-micro-control-system/assets/102282348/0a99aa31-5a80-47ab-a8da-104dbc5ca879)
  2
  ![image](https://github.com/COFFEEOfficial/XIDIAN-micro-control-system/assets/102282348/f1de90de-fdca-443c-818b-eb9cf490aed1)


3       ![image](https://github.com/COFFEEOfficial/XIDIAN-micro-control-system/assets/102282348/202ac6b9-9684-45b1-ba3d-e540f68dfd98)
                          

4
  ![image](https://github.com/COFFEEOfficial/XIDIAN-micro-control-system/assets/102282348/2049bc49-741f-4345-b0f6-be7dc5563845)

具体代码：
module servo(
input clk,         //时钟 50MHz
input sw1,          //调速按键1
input sw2,          //调速按键2
input sw3,
output pwm,    //pwm输出
output ABC
);

reg [19:0] pwm_val;       //占空比计数值
reg [19:0] ABC_val;       //占空比计数值
reg [3:0] speed = 4'd2;   //转向角度选择
reg [3:0] speed1 = 4'd2;   //转向角度选择
reg pwm;
reg ABC;
wire sw11;
wire sw22;
parameter max_cnt=1000_000; 
reg [19:0] clk_cnt;

assign sw11=sw1;
assign sw22=sw2;
always @(posedge clk)begin //产生20ms周期计时
    if(clk_cnt==max_cnt)begin
    clk_cnt=20'b0;
    end
    else begin
    clk_cnt=clk_cnt+1'b1;
    end
end

//PWM产生模块
always @ (posedge clk) begin
    if(clk_cnt < pwm_val) begin           //如果在pwm_val内，输出高电平
    pwm <=1'b1;
    end
    else begin                   //如果超出pwm_val，则输出低电平
    pwm <=1'b0;                  //输出低电平
    end
    case(speed)
        1: pwm_val <= 20'd50_000; //占空比为5% 1ms
        2: pwm_val <= 20'd100_000; //10% 2ms
        3: pwm_val <= 20'd25_000;
        4: pwm_val <= 20'd75_000;
        5: pwm_val <= 20'd1_000;
        default: pwm_val <= 20'd1_000;
    endcase
end

//ABC产生模块
always @ (posedge clk) begin
   
    if(clk_cnt < ABC_val) begin           //如果在pwm_val内，输出高电平
    ABC <=1'b1;
    end
    else begin                   //如果超出pwm_val，则输出低电平
    ABC <=1'b0;                  //输出低电平
    end
    case(speed1)
        1: ABC_val <= 20'd50_000; //占空比为5% 1ms
        2: ABC_val <= 20'd100_000; //10% 2ms
        3: ABC_val <= 20'd25_000;
        4: ABC_val <= 20'd75_000;
        5: ABC_val <= 20'd1_000;
        default: ABC_val <= 20'd1_000;
    endcase
end

//开关调速
always @ (posedge clk) begin
    if(~sw11 && ~sw22 ) begin
        speed <= 4'd5;
    end    
    if(~sw11 && sw22 ) begin
        speed <= 4'd1;
    end      
    if(sw11 && ~sw22 ) begin
        speed <= 4'd2;
    end            
    if(sw11 && sw22 ) begin
         speed <= 4'd5;
    end          
        if(~sw3 ) begin
            speed1 <= 4'd1;
        end    
        if(~sw3) begin
            speed1 <= 4'd1;
        end      
        if(~sw3 ) begin
            speed1 <= 4'd1;
        end            
        if(sw3 ) begin
             speed1 <= 4'd2;
        end              
end                         
endmodule
