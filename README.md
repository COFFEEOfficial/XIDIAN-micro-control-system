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



Java
Main：
package com.example.sang.testserial;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.ScrollView;
import android.widget.TextView;
import android.widget.Toast;
import java.util.Timer;
import java.util.TimerTask;
import com.friendlyarm.FriendlyThings.HardwareControler;

import android.os.Handler;
import android.os.Message;
import android.content.Intent;


public class MainActivity extends Activity implements OnClickListener {

    private static final String TAG = "SerialPort";
    private TextView fromTextView = null;//
    private final int MAXLINES = 200;
    private StringBuilder remoteData = new StringBuilder(256 * MAXLINES);

    // NanoPC-T4 UART4
    private String devName = "/dev/ttyAMA3";
    private int speed = 115200;
    private int dataBits = 8;
    private int stopBits = 1;
    private int devfd = -1;
    private String str = "";
    private String str1 = "";

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);//这里是需要改的

        String winTitle = devName + "," + speed + "," + dataBits + "," + stopBits;
        setTitle(winTitle);

        ((Button)findViewById(R.id.sendButton0)).setOnClickListener(this);
        ((Button)findViewById(R.id.sendButton3)).setOnClickListener(this);
        ((Button)findViewById(R.id.sendButton4)).setOnClickListener(this);

        fromTextView = (TextView)findViewById(R.id.fromTextView);



        /* no focus when begin */

        devfd = HardwareControler.openSerialPort( devName, speed, dataBits, stopBits );
        if (devfd >= 0) {
            timer.schedule(task, 0, 500);
        } else {
            devfd = -1;
        }
    }

    @Override
    public void onClick(View v) {
        //跳转函数
        if (v.getId() == R.id.sendButton4) {

            //跳转到登录界面
            Intent intent = new Intent(MainActivity.this,Main2Activity.class);
            startActivity(intent);
            finish();
        }//结束
        if (v.getId() == R.id.sendButton0) {

            //跳转到英文界面
            Intent intent = new Intent(MainActivity.this, Main4Activity.class);
            startActivity(intent);
            finish();
        }//结束
        if (v.getId() == R.id.sendButton3) {

            //跳转到用户界面
            Intent intent = new Intent(MainActivity.this, Main7Activity.class);
            startActivity(intent);
            finish();
        }//结束
    }
    //
    private final int BUFSIZE = 512;
    private byte[] buf = new byte[BUFSIZE];
    private Timer timer = new Timer();
    private Handler handler = new Handler() {
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 1:
                    if (HardwareControler.select(devfd, 0, 0) == 1) {
                        int retSize = HardwareControler.read(devfd, buf, BUFSIZE);
                        if (retSize > 0) {
                            String str = new String(buf, 0, retSize);
                            //
                            remoteData.append(str);

                            Toast.makeText(MainActivity.this, str, Toast.LENGTH_SHORT).show();

                            if (fromTextView.getLineCount() > MAXLINES) {
                                int nLineCount = fromTextView.getLineCount();
                                int i = 0;
                                for (i = 0; i < remoteData.length(); i++) {
                                    if (remoteData.charAt(i) == '\n') {
                                        nLineCount--;
                                        if (nLineCount <= MAXLINES) {
                                            break;
                                        }
                                    }
                                }
                                remoteData.delete(0, i);
                                fromTextView.setText(remoteData.toString());
                            } else {
                                fromTextView.append(str);
                            }
                            ((ScrollView) findViewById(R.id.scroolView)).fullScroll(View.FOCUS_DOWN);
                            //

                            //
                        }
                    }
                    break;
            }
            super.handleMessage(msg);
        }
    };

    private TimerTask task = new TimerTask() {
        public void run() {
            Message message = new Message();
            message.what = 1;
            handler.sendMessage(message);
        }
    };

    public void onDestroy() {
        timer.cancel();
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onDestroy();
    }
    public void onPause(){
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onPause();
    }
    public void onStop(){
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onStop();
    }
}
Main2：
package com.example.sang.testserial;

import android.support.v7.app.AppCompatActivity;

import android.os.Bundle;
import android.widget.Button;
import android.view.View;
import android.widget.EditText;
import android.content.Intent;
import android.widget.Toast;

public class Main2Activity extends AppCompatActivity implements View.OnClickListener {
    //申明控件
    private Button mybuttondenglu;
    private EditText mEtUser;
    private EditText mEtPassword;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);
        //找到控件
        mybuttondenglu = findViewById(R.id.but0);
        mEtUser = findViewById(R.id.edit1);
        mEtPassword = findViewById(R.id.edit2);

        mybuttondenglu.setOnClickListener(this);

    }
    public void onClick(View v){
        //需要获取输入的用户名和密码
        String username = mEtUser.getText().toString();
        String password = mEtPassword.getText().toString();
        Intent intent = null;

        //假设正确的账号为2101055；密码为13579246810.
        if(username.equals("55")&&password.equals("123456")){
            intent = new Intent(Main2Activity.this,Main3Activity.class);
            startActivity(intent);
            finish();
        }
        else{
            Toast.makeText(Main2Activity.this, "登录失败", Toast.LENGTH_SHORT).show();
        }

    }

}
Main3：
package com.example.sang.testserial;

import android.app.Activity;
import android.content.res.Configuration;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ScrollView;
import android.widget.TextView;
import android.util.Log;
import android.text.Html;
import android.widget.Toast;
import java.util.Timer;
import java.util.TimerTask;
import com.friendlyarm.FriendlyThings.HardwareControler;
import com.friendlyarm.FriendlyThings.BoardType;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.content.Context;
import android.content.Intent;


public class Main3Activity extends Activity implements OnClickListener {//这里是需要改的
    //定义控件

    //别人的代码
    private static final String TAG = "SerialPort";
    private final int MAXLINES = 200;
    private TextView fromTextView = null;//
    private StringBuilder remoteData = new StringBuilder(256 * MAXLINES);
    //private变量自己定义

    // NanoPC-T4 UART4
    private String devName = "/dev/ttyAMA3";
    private int speed = 115200;
    private int dataBits = 8;
    private int stopBits = 1;
    private int devfd = -1;
    private String str = "";
    //这部分是波特率

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main3);//这里是需要改的

        String winTitle = devName + "," + speed + "," + dataBits + "," + stopBits;
        setTitle(winTitle);

        fromTextView = (TextView)findViewById(R.id.fromTextView);
        //1.得到控件
        //找控件需要做两步，第一步设置id，第二步通过findViewById找到控件

        ((Button)findViewById(R.id.btn_duoji1)).setOnClickListener(this);
        ((Button)findViewById(R.id.btn_duoji2)).setOnClickListener(this);
        ((Button)findViewById(R.id.btn_color)).setOnClickListener(this);
        //((Button)findViewById(R.id.btn_margin)).setOnClickListener(this);

        //if语句判断
        devfd = HardwareControler.openSerialPort( devName, speed, dataBits, stopBits );
        if (devfd >= 0) {
            timer.schedule(task, 0, 500);
        } else {
            devfd = -1;
        }
        //if语句判断控件是否被点击
    }

    //点击某个按钮 每个按钮有不同的函数分别得到不同的结果
    @Override
    public void onClick(View v) {
        //舵机1的测试函数
        if (v.getId() == R.id.btn_duoji1) {
            String str_0 = "C";//可更改发送信息
            if (str_0.length() > 0) {
                int ret = HardwareControler.write(devfd, str_0.getBytes());
                if (ret > 0) {
                    remoteData.append(str_0);
                }
            }
            remoteData.append(str_0);
        }//舵机1测试函数完毕
        //舵机2测试函数
        if (v.getId() == R.id.btn_duoji2) {
            String str_0 = "D";//可更改发送信息
            if (str_0.length() > 0) {
                int ret = HardwareControler.write(devfd, str_0.getBytes());
                if (ret > 0) {
                    remoteData.append(str_0);
                }
            }
            remoteData.append(str_0);
        }//舵机2测试函数完毕

        //颜色传感器数据发送界面
        if (v.getId() == R.id.btn_color) {
            String str_0 = "A";//可更改发送信息
            if (str_0.length() > 0) {
                int ret = HardwareControler.write(devfd, str_0.getBytes());
                if (ret > 0) {
                    remoteData.append(str_0);
                }
            }
            remoteData.append(str_0);
        }//颜色传感器发送界面结束
        //距离传感器数据发送界面
        /*if (v.getId() == R.id.btn_margin) {
            String str_0 = "d";//可更改发送信息
            if (str_0.length() > 0) {
                int ret = HardwareControler.write(devfd, str_0.getBytes());
                if (ret > 0) {
                    remoteData.append(str_0);
                }
            }
            remoteData.append(str_0);
        }*/
        //距离传感器数据发送界面结束
    }

    //上面部分的截止

    //上位机接受MCU发送的信息并做出回应

    private final int BUFSIZE = 512;
    private byte[] buf = new byte[BUFSIZE];
    private Timer timer = new Timer();
    private Handler handler = new Handler() {
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 1:
                    if (HardwareControler.select(devfd, 0, 0) == 1) {
                        int retSize = HardwareControler.read(devfd, buf, BUFSIZE);
                        if (retSize > 0) {
                            String str = new String(buf, 0, retSize);
                            remoteData.append(str);
                            //
                            Toast.makeText(Main3Activity.this, str, Toast.LENGTH_SHORT).show();
                            if (fromTextView.getLineCount() > MAXLINES) {
                                int nLineCount = fromTextView.getLineCount();
                                int i = 0;
                                for (i = 0; i < remoteData.length(); i++) {
                                    if (remoteData.charAt(i) == '\n') {
                                        nLineCount--;
                                        if (nLineCount <= MAXLINES) {
                                            break;
                                        }
                                    }
                                }
                                remoteData.delete(0, i);
                                fromTextView.setText(remoteData.toString());
                            } else {
                                fromTextView.append(str);
                            }
                            ((ScrollView) findViewById(R.id.scroolView)).fullScroll(View.FOCUS_DOWN);
                            //
                        }
                    }
                    break;
            }
            super.handleMessage(msg);
        }
    };
    //上位机部分回应MCU部分结束

    private TimerTask task = new TimerTask() {
        public void run() {
            Message message = new Message();
            message.what = 1;
            handler.sendMessage(message);
        }
    };

    public void onDestroy() {
        timer.cancel();
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onDestroy();
    }
    public void onPause(){
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onPause();
    }
    public void onStop(){
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onStop();
    }
}
Main4：
package com.example.sang.testserial;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.ScrollView;
import android.widget.TextView;
import android.widget.Toast;
import java.util.Timer;
import java.util.TimerTask;
import com.friendlyarm.FriendlyThings.HardwareControler;

import android.os.Handler;
import android.os.Message;
import android.content.Intent;


public class Main4Activity extends Activity implements OnClickListener {

    private static final String TAG = "SerialPort";
    private TextView fromTextView = null;//
    private final int MAXLINES = 200;
    private StringBuilder remoteData = new StringBuilder(256 * MAXLINES);

    // NanoPC-T4 UART4
    private String devName = "/dev/ttyAMA3";
    private int speed = 115200;
    private int dataBits = 8;
    private int stopBits = 1;
    private int devfd = -1;
    private String str = "";
    private String str1 = "";

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main4);//这里是需要改的

        String winTitle = devName + "," + speed + "," + dataBits + "," + stopBits;
        setTitle(winTitle);

        ((Button)findViewById(R.id.sendButton0e)).setOnClickListener(this);
        ((Button)findViewById(R.id.sendButton3e)).setOnClickListener(this);
        ((Button)findViewById(R.id.sendButton4e)).setOnClickListener(this);

        fromTextView = (TextView)findViewById(R.id.fromTextViewe);



        /* no focus when begin */

        devfd = HardwareControler.openSerialPort( devName, speed, dataBits, stopBits );
        if (devfd >= 0) {
            timer.schedule(task, 0, 500);
        } else {
            devfd = -1;
        }
    }

    @Override
    public void onClick(View v) {
        //跳转函数
        if (v.getId() == R.id.sendButton4e) {

            //跳转到登录界面
            Intent intent = new Intent(Main4Activity.this,Main5Activity.class);
            startActivity(intent);
            finish();
        }//结束
        if (v.getId() == R.id.sendButton0e) {

            //跳转到中文界面
            Intent intent = new Intent(Main4Activity.this, MainActivity.class);
            startActivity(intent);
            finish();
        }//结束
        if (v.getId() == R.id.sendButton3e) {

            //跳转到用户界面
            Intent intent = new Intent(Main4Activity.this, Main8Activity.class);
            startActivity(intent);
            finish();
        }//结束
    }
    //
    private final int BUFSIZE = 512;
    private byte[] buf = new byte[BUFSIZE];
    private Timer timer = new Timer();
    private Handler handler = new Handler() {
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 1:
                    if (HardwareControler.select(devfd, 0, 0) == 1) {
                        int retSize = HardwareControler.read(devfd, buf, BUFSIZE);
                        if (retSize > 0) {
                            String str = new String(buf, 0, retSize);
                            //
                            remoteData.append(str);

                            Toast.makeText(Main4Activity.this, str, Toast.LENGTH_SHORT).show();

                            if (fromTextView.getLineCount() > MAXLINES) {
                                int nLineCount = fromTextView.getLineCount();
                                int i = 0;
                                for (i = 0; i < remoteData.length(); i++) {
                                    if (remoteData.charAt(i) == '\n') {
                                        nLineCount--;
                                        if (nLineCount <= MAXLINES) {
                                            break;
                                        }
                                    }
                                }
                                remoteData.delete(0, i);
                                fromTextView.setText(remoteData.toString());
                            } else {
                                fromTextView.append(str);
                            }
                            ((ScrollView) findViewById(R.id.scroolViewe)).fullScroll(View.FOCUS_DOWN);
                            //

                            //
                        }
                    }
                    break;
            }
            super.handleMessage(msg);
        }
    };

    private TimerTask task = new TimerTask() {
        public void run() {
            Message message = new Message();
            message.what = 1;
            handler.sendMessage(message);
        }
    };

    public void onDestroy() {
        timer.cancel();
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onDestroy();
    }
    public void onPause(){
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onPause();
    }
    public void onStop(){
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onStop();
    }
}
Main5：
package com.example.sang.testserial;

import android.support.v7.app.AppCompatActivity;

import android.os.Bundle;
import android.widget.Button;
import android.view.View;
import android.widget.EditText;
import android.content.Intent;
import android.widget.Toast;

public class Main5Activity extends AppCompatActivity implements View.OnClickListener {
    //申明控件
    private Button mybuttondenglue;
    private EditText mEtUsere;
    private EditText mEtPassworde;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main5);
        //找到控件
        mybuttondenglue = findViewById(R.id.but0e);
        mEtUsere = findViewById(R.id.edit1e);
        mEtPassworde = findViewById(R.id.edit2e);

        mybuttondenglue.setOnClickListener(this);

    }
    public void onClick(View v){
        //需要获取输入的用户名和密码
        String username = mEtUsere.getText().toString();
        String password = mEtPassworde.getText().toString();
        Intent intent = null;

        //假设正确的账号为2101055；密码为13579246810.
        if(username.equals("55")&&password.equals("123456")){
            intent = new Intent(Main5Activity.this,Main6Activity.class);
            startActivity(intent);
            finish();
        }
        else{
            Toast.makeText(Main5Activity.this, "Fail To Login", Toast.LENGTH_SHORT).show();
        }

    }

}
Main6：
package com.example.sang.testserial;

import android.app.Activity;
import android.content.res.Configuration;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ScrollView;
import android.widget.TextView;
import android.util.Log;
import android.text.Html;
import android.widget.Toast;
import java.util.Timer;
import java.util.TimerTask;
import com.friendlyarm.FriendlyThings.HardwareControler;
import com.friendlyarm.FriendlyThings.BoardType;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.content.Context;
import android.content.Intent;


public class Main6Activity extends Activity implements OnClickListener {//这里是需要改的
    //定义控件

    //别人的代码
    private static final String TAG = "SerialPort";
    private final int MAXLINES = 200;
    private TextView fromTextView = null;//
    private StringBuilder remoteData = new StringBuilder(256 * MAXLINES);
    //private变量自己定义

    // NanoPC-T4 UART4
    private String devName = "/dev/ttyAMA3";
    private int speed = 115200;
    private int dataBits = 8;
    private int stopBits = 1;
    private int devfd = -1;
    private String str = "";
    //这部分是波特率

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main6);//这里是需要改的

        String winTitle = devName + "," + speed + "," + dataBits + "," + stopBits;
        setTitle(winTitle);

        fromTextView = (TextView)findViewById(R.id.fromTextViewe);
        //1.得到控件
        //找控件需要做两步，第一步设置id，第二步通过findViewById找到控件

        ((Button)findViewById(R.id.btn_duoji1e)).setOnClickListener(this);
        ((Button)findViewById(R.id.btn_duoji2e)).setOnClickListener(this);
        ((Button)findViewById(R.id.btn_colore)).setOnClickListener(this);
        //((Button)findViewById(R.id.btn_margine)).setOnClickListener(this);

        //if语句判断
        devfd = HardwareControler.openSerialPort( devName, speed, dataBits, stopBits );
        if (devfd >= 0) {
            timer.schedule(task, 0, 500);
        } else {
            devfd = -1;
        }
        //if语句判断控件是否被点击
    }

    //点击某个按钮 每个按钮有不同的函数分别得到不同的结果
    @Override
    public void onClick(View v) {
        //舵机1的测试函数
        if (v.getId() == R.id.btn_duoji1e) {
            String str_0 = "C";//可更改发送信息
            if (str_0.length() > 0) {
                int ret = HardwareControler.write(devfd, str_0.getBytes());
                if (ret > 0) {
                    remoteData.append(str_0);
                }
            }
            remoteData.append(str_0);
        }//舵机1测试函数完毕
        //舵机2测试函数
        if (v.getId() == R.id.btn_duoji2e) {
            String str_0 = "D";//可更改发送信息
            if (str_0.length() > 0) {
                int ret = HardwareControler.write(devfd, str_0.getBytes());
                if (ret > 0) {
                    remoteData.append(str_0);
                }
            }
            remoteData.append(str_0);
        }//舵机2测试函数完毕

        //颜色传感器数据发送界面
        if (v.getId() == R.id.btn_colore) {
            String str_0 = "A";//可更改发送信息
            if (str_0.length() > 0) {
                int ret = HardwareControler.write(devfd, str_0.getBytes());
                if (ret > 0) {
                    remoteData.append(str_0);
                }
            }
            remoteData.append(str_0);

        }//颜色传感器发送界面结束
        //距离传感器数据发送界面
        /*if (v.getId() == R.id.btn_margine) {
            String str_0 = "3";//可更改发送信息
            if (str_0.length() > 0) {
                int ret = HardwareControler.write(devfd, str_0.getBytes());
                if (ret > 0) {
                    remoteData.append(str_0);
                }
            }
            remoteData.append(str_0);
        }*/
        //距离传感器数据发送界面结束
    }

    //上面部分的截止

    //上位机接受MCU发送的信息并做出回应

    private final int BUFSIZE = 512;
    private byte[] buf = new byte[BUFSIZE];
    private Timer timer = new Timer();
    private Handler handler = new Handler() {
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 1:
                    if (HardwareControler.select(devfd, 0, 0) == 1) {
                        int retSize = HardwareControler.read(devfd, buf, BUFSIZE);
                        if (retSize > 0) {
                            String str = new String(buf, 0, retSize);
                            remoteData.append(str);
                            //
                            Toast.makeText(Main6Activity.this, str, Toast.LENGTH_SHORT).show();
                            if (fromTextView.getLineCount() > MAXLINES) {
                                int nLineCount = fromTextView.getLineCount();
                                int i = 0;
                                for (i = 0; i < remoteData.length(); i++) {
                                    if (remoteData.charAt(i) == '\n') {
                                        nLineCount--;
                                        if (nLineCount <= MAXLINES) {
                                            break;
                                        }
                                    }
                                }
                                remoteData.delete(0, i);
                                fromTextView.setText(remoteData.toString());
                            } else {
                                fromTextView.append(str);
                            }
                            ((ScrollView) findViewById(R.id.scroolViewe)).fullScroll(View.FOCUS_DOWN);
                            //
                        }
                    }
                    break;
            }
            super.handleMessage(msg);
        }
    };
    //上位机部分回应MCU部分结束

    private TimerTask task = new TimerTask() {
        public void run() {
            Message message = new Message();
            message.what = 1;
            handler.sendMessage(message);
        }
    };

    public void onDestroy() {
        timer.cancel();
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onDestroy();
    }
    public void onPause(){
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onPause();
    }
    public void onStop(){
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onStop();
    }
}
Main7：
package com.example.sang.testserial;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.ScrollView;
import android.widget.TextView;
import android.widget.Toast;
import java.util.Timer;
import java.util.TimerTask;
import com.friendlyarm.FriendlyThings.HardwareControler;

import android.os.Handler;
import android.os.Message;
import android.content.Intent;


public class Main7Activity extends Activity implements OnClickListener {

    private static final String TAG = "SerialPort";
    private TextView fromTextView = null;//
    private final int MAXLINES = 200;
    private StringBuilder remoteData = new StringBuilder(256 * MAXLINES);

    // NanoPC-T4 UART4
    private String devName = "/dev/ttyAMA3";
    private int speed = 115200;
    private int dataBits = 8;
    private int stopBits = 1;
    private int devfd = -1;
    private String str = "";
    private String str1 = "";

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main7);//这里是需要改的

        String winTitle = devName + "," + speed + "," + dataBits + "," + stopBits;
        setTitle(winTitle);

        ((Button)findViewById(R.id.sendButton20)).setOnClickListener(this);


        fromTextView = (TextView)findViewById(R.id.fromTextView);



        /* no focus when begin */

        devfd = HardwareControler.openSerialPort( devName, speed, dataBits, stopBits );
        if (devfd >= 0) {
            timer.schedule(task, 0, 500);
        } else {
            devfd = -1;
        }
    }

    @Override
    public void onClick(View v) {
        //运行模式函数函数
        if (v.getId() == R.id.sendButton20) {
            String str_0 = "A";//可更改发送信息
            if (str_0.length() > 0) {
                int ret = HardwareControler.write(devfd, str_0.getBytes());
                if (ret > 0) {
                    remoteData.append(str_0);
                }
            }
            remoteData.append(str_0);
        }
        //
    }
    //
    private final int BUFSIZE = 512;
    private byte[] buf = new byte[BUFSIZE];
    private Timer timer = new Timer();
    private Handler handler = new Handler() {
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 1:
                    if (HardwareControler.select(devfd, 0, 0) == 1) {
                        int retSize = HardwareControler.read(devfd, buf, BUFSIZE);
                        if (retSize > 0) {
                            String str = new String(buf, 0, retSize);
                            //
                            remoteData.append(str);

                            Toast.makeText(Main7Activity.this, str, Toast.LENGTH_SHORT).show();

                            if (fromTextView.getLineCount() > MAXLINES) {
                                int nLineCount = fromTextView.getLineCount();
                                int i = 0;
                                for (i = 0; i < remoteData.length(); i++) {
                                    if (remoteData.charAt(i) == '\n') {
                                        nLineCount--;
                                        if (nLineCount <= MAXLINES) {
                                            break;
                                        }
                                    }
                                }
                                remoteData.delete(0, i);
                                fromTextView.setText(remoteData.toString());
                            } else {
                                fromTextView.append(str);
                            }
                            ((ScrollView) findViewById(R.id.scroolView)).fullScroll(View.FOCUS_DOWN);
                            //

                            //
                        }
                    }
                    break;
            }
            super.handleMessage(msg);
        }
    };

    private TimerTask task = new TimerTask() {
        public void run() {
            Message message = new Message();
            message.what = 1;
            handler.sendMessage(message);
        }
    };

    public void onDestroy() {
        timer.cancel();
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onDestroy();
    }
    public void onPause(){
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onPause();
    }
    public void onStop(){
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onStop();
    }
}
Main8：
package com.example.sang.testserial;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.ScrollView;
import android.widget.TextView;
import android.widget.Toast;
import java.util.Timer;
import java.util.TimerTask;
import com.friendlyarm.FriendlyThings.HardwareControler;

import android.os.Handler;
import android.os.Message;
import android.content.Intent;


public class Main8Activity extends Activity implements OnClickListener {

    private static final String TAG = "SerialPort";
    private TextView fromTextView = null;//
    private final int MAXLINES = 200;
    private StringBuilder remoteData = new StringBuilder(256 * MAXLINES);

    // NanoPC-T4 UART4
    private String devName = "/dev/ttyAMA3";
    private int speed = 115200;
    private int dataBits = 8;
    private int stopBits = 1;
    private int devfd = -1;
    private String str = "";
    private String str1 = "";

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main8);//这里是需要改的

        String winTitle = devName + "," + speed + "," + dataBits + "," + stopBits;
        setTitle(winTitle);

        ((Button)findViewById(R.id.sendButton20e)).setOnClickListener(this);


        fromTextView = (TextView)findViewById(R.id.fromTextViewe);



        /* no focus when begin */

        devfd = HardwareControler.openSerialPort( devName, speed, dataBits, stopBits );
        if (devfd >= 0) {
            timer.schedule(task, 0, 500);
        } else {
            devfd = -1;
        }
    }

    @Override
    public void onClick(View v) {
        //运行模式函数函数
        if (v.getId() == R.id.sendButton20e) {
            String str_0 = "A";//可更改发送信息
            if (str_0.length() > 0) {
                int ret = HardwareControler.write(devfd, str_0.getBytes());
                if (ret > 0) {
                    remoteData.append(str_0);
                }
            }
            remoteData.append(str_0);
        }
        //
    }
    //
    private final int BUFSIZE = 512;
    private byte[] buf = new byte[BUFSIZE];
    private Timer timer = new Timer();
    private Handler handler = new Handler() {
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 1:
                    if (HardwareControler.select(devfd, 0, 0) == 1) {
                        int retSize = HardwareControler.read(devfd, buf, BUFSIZE);
                        if (retSize > 0) {
                            String str = new String(buf, 0, retSize);
                            //
                            remoteData.append(str);

                            Toast.makeText(Main8Activity.this, str, Toast.LENGTH_SHORT).show();

                            if (fromTextView.getLineCount() > MAXLINES) {
                                int nLineCount = fromTextView.getLineCount();
                                int i = 0;
                                for (i = 0; i < remoteData.length(); i++) {
                                    if (remoteData.charAt(i) == '\n') {
                                        nLineCount--;
                                        if (nLineCount <= MAXLINES) {
                                            break;
                                        }
                                    }
                                }
                                remoteData.delete(0, i);
                                fromTextView.setText(remoteData.toString());
                            } else {
                                fromTextView.append(str);
                            }
                            ((ScrollView) findViewById(R.id.scroolViewe)).fullScroll(View.FOCUS_DOWN);
                            //

                            //
                        }
                    }
                    break;
            }
            super.handleMessage(msg);
        }
    };

    private TimerTask task = new TimerTask() {
        public void run() {
            Message message = new Message();
            message.what = 1;
            handler.sendMessage(message);
        }
    };

    public void onDestroy() {
        timer.cancel();
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onDestroy();
    }
    public void onPause(){
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onPause();
    }
    public void onStop(){
        if (devfd != -1) {
            HardwareControler.close(devfd);
            devfd = -1;
        }
        super.onStop();
    }
}
xml文件：
Main：
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_horizontal"
    android:orientation="vertical"
    android:padding="10dp"
    tools:context=".MainActivity"
    android:background="@drawable/main1"
    >


    <Button
        android:id="@+id/sendButton0"
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:layout_marginTop="80dp"
        android:background="@drawable/rounded_button_little_green"
        android:text="欢迎来到智能药箱（中/英）"
        android:textSize="25sp" />
    <Button
        android:id="@+id/sendButton3"
        android:layout_width="400dp"
        android:layout_height="90dp"
        android:layout_marginTop="140dp"
        android:background="@drawable/rounded_button_blue"
        android:text="用户模式"
        android:textSize="20sp" />

   <!-- <Button
        android:id="@+id/sendButton1"
        android:layout_width="350dp"
        android:layout_height="90dp"
        android:layout_marginTop="10dp"
        android:background="@drawable/rounded_button_yellow"
        android:text="黄色药品"
        android:textSize="20sp" />

    <Button
        android:id="@+id/sendButton2"
        android:layout_width="350dp"
        android:layout_height="90dp"
        android:layout_marginTop="10dp"
        android:background="@drawable/rounded_button_green"
        android:text="绿色药品"
        android:textSize="20sp" />

    <Button
        android:id="@+id/sendButton3"
        android:layout_width="350dp"
        android:layout_height="90dp"
        android:layout_marginTop="10dp"
        android:background="@drawable/rounded_button_blue"
        android:text="蓝色药品"
        android:textSize="20sp" />-->

    <Button
        android:id="@+id/sendButton4"
        android:layout_width="400dp"
        android:layout_height="90dp"
        android:layout_marginTop="30dp"
        android:background="@drawable/rounded_button_red"
        android:text="维护模式"
        android:textSize="20sp" />


    <!--<TextView
        android:id="@+id/label1"
        android:layout_width="400dp"
        android:layout_height="80dp"
        android:text="药品类型"
        android:gravity="center"
        android:textSize="25sp" />-->

    <com.friendlyarm.Utils.BorderScrollView
        android:id="@+id/scroolView"
        android:layout_width="match_parent"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:layout_marginTop="30dp"
        android:background="@color/colorPrimary">

        <TextView
            android:id="@+id/fromTextView"
            android:layout_width="match_parent"
            android:layout_height="fill_parent"
            android:background="@color/colorPrimary"
            android:gravity="top"
            android:singleLine="false"
            android:textColor="#000000" />
    </com.friendlyarm.Utils.BorderScrollView>

</LinearLayout>
Main2：
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center_horizontal"
    android:padding="10dp"
    tools:context=".Main2Activity"
    android:background="@drawable/main1"
    >


    <TextView
        android:id="@+id/sendButton5"
        android:layout_width="400dp"
        android:layout_height="80dp"
        android:text="欢迎来到智能药箱维护模式"
        android:gravity="center"
        android:textSize="30sp"
        android:background="@drawable/rounded_button_little_green"
        />




    <EditText
        android:id="@+id/edit1"
        android:layout_width="400dp"
        android:layout_height="100dp"
        android:layout_marginTop="100dp"
        android:textSize="16sp"
        android:hint="用户名"
        android:maxLines="1"

        />
    <EditText
        android:id="@+id/edit2"
        android:layout_width="400dp"
        android:layout_height="100dp"
        android:layout_marginTop="10dp"
        android:textSize="16sp"
        android:hint="密码"
        android:maxLines="1"
        android:inputType="textPassword"
        />
    <Button
        android:id="@+id/but0"
        android:layout_width="400dp"
        android:layout_height="100dp"
        android:text="登录"
        android:background="@drawable/rounded_button_color_primary"
        />




</LinearLayout>
Main3：
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"

    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="10dp"
    android:gravity="center_horizontal"
    tools:context=".Main3Activity"
    android:background="@drawable/main1"
    >


    <TextView
        android:id="@+id/tv_func_ceshi"
        android:layout_width="300dp"
        android:layout_height="50dp"
        android:text="请选择测试项目"
        android:layout_marginTop="30dp"
        android:textSize="25sp"
        android:gravity="center"
        android:background="@drawable/rounded_button_little_green"
        />

    <Button
        android:id="@+id/btn_duoji1"
        android:layout_width="300dp"
        android:layout_height="100dp"
        android:text="舵机1旋转测试"
        android:layout_marginTop="60dp"
        android:textSize="25sp"
        android:layout_gravity="center"
        android:background="@drawable/rounded_button_little_blue"
        />
    <Button
        android:id="@+id/btn_duoji2"
        android:layout_width="300dp"
        android:layout_height="100dp"
        android:text="舵机2旋转测试"
        android:textSize="25sp"
        android:layout_marginTop="10dp"
        android:layout_gravity="center"
        android:background="@drawable/rounded_button_little_blue"
        />
    <Button
        android:id="@+id/btn_color"
        android:layout_width="300dp"
        android:layout_height="100dp"
        android:text="传感器测试"
        android:layout_marginTop="10dp"
        android:textSize="25sp"
        android:layout_gravity="center"
        android:background="@drawable/rounded_button_little_blue"
        />
    <!--<Button
        android:id="@+id/btn_margin"
        android:layout_width="300dp"
        android:layout_height="100dp"
        android:text="距离传感器测试"
        android:textSize="25sp"
        android:layout_marginTop="10dp"
        android:layout_gravity="center"
        android:background="@drawable/rounded_button_little_blue"
        />-->
    <com.friendlyarm.Utils.BorderScrollView
        android:id="@+id/scroolView"
        android:layout_width="match_parent"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:layout_marginTop="20dp"
        android:background="@color/colorPrimary">

        <TextView
            android:id="@+id/fromTextView"
            android:layout_width="match_parent"
            android:layout_height="fill_parent"
            android:background="@color/colorPrimary"
            android:gravity="top"
            android:singleLine="false"
            android:textColor="#000000" />
    </com.friendlyarm.Utils.BorderScrollView>

</LinearLayout>
Main4：
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_horizontal"
    android:orientation="vertical"
    android:padding="10dp"
    tools:context=".Main4Activity"
    android:background="@drawable/main1"
    >


    <Button
        android:id="@+id/sendButton0e"
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:layout_marginTop="80dp"
        android:background="@drawable/rounded_button_little_green"
        android:text="The Smart Medicine Box!\n(Chinese/English)"
        android:textAllCaps="false"
        android:textSize="25sp" />
    <Button
        android:id="@+id/sendButton3e"
        android:layout_width="400dp"
        android:layout_height="90dp"
        android:layout_marginTop="140dp"
        android:background="@drawable/rounded_button_blue"
        android:text="User Mode"
        android:textAllCaps="false"
        android:textSize="20sp" />

    <!-- <Button
         android:id="@+id/sendButton1"
         android:layout_width="350dp"
         android:layout_height="90dp"
         android:layout_marginTop="10dp"
         android:background="@drawable/rounded_button_yellow"
         android:text="黄色药品"
         android:textSize="20sp" />

     <Button
         android:id="@+id/sendButton2"
         android:layout_width="350dp"
         android:layout_height="90dp"
         android:layout_marginTop="10dp"
         android:background="@drawable/rounded_button_green"
         android:text="绿色药品"
         android:textSize="20sp" />

     <Button
         android:id="@+id/sendButton3"
         android:layout_width="350dp"
         android:layout_height="90dp"
         android:layout_marginTop="10dp"
         android:background="@drawable/rounded_button_blue"
         android:text="蓝色药品"
         android:textSize="20sp" />-->

    <Button
        android:id="@+id/sendButton4e"
        android:layout_width="400dp"
        android:layout_height="90dp"
        android:layout_marginTop="30dp"
        android:background="@drawable/rounded_button_red"
        android:text="“Maintenance Mode"
        android:textAllCaps="false"
        android:textSize="20sp" />


    <!--<TextView
        android:id="@+id/label1"
        android:layout_width="400dp"
        android:layout_height="80dp"
        android:text="药品类型"
        android:gravity="center"
        android:textSize="25sp" />-->

    <com.friendlyarm.Utils.BorderScrollView
        android:id="@+id/scroolViewe"
        android:layout_width="match_parent"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:layout_marginTop="30dp"
        android:background="@color/colorPrimary">

        <TextView
            android:id="@+id/fromTextViewe"
            android:layout_width="match_parent"
            android:layout_height="fill_parent"
            android:background="@color/colorPrimary"
            android:gravity="top"
            android:singleLine="false"
            android:textColor="#000000" />
    </com.friendlyarm.Utils.BorderScrollView>

</LinearLayout>
Main5：
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center_horizontal"
    android:padding="10dp"
    tools:context=".Main5Activity"
    android:background="@drawable/main1"
    >


    <TextView
        android:id="@+id/sendButton5e"
        android:layout_width="400dp"
        android:layout_height="80dp"
        android:text="Maintenance Mode"
        android:gravity="center"
        android:textSize="30sp"
        android:background="@drawable/rounded_button_little_green"
        />




    <EditText
        android:id="@+id/edit1e"
        android:layout_width="400dp"
        android:layout_height="100dp"
        android:layout_marginTop="100dp"
        android:textSize="16sp"
        android:hint="User'name"
        android:maxLines="1"

        />
    <EditText
        android:id="@+id/edit2e"
        android:layout_width="400dp"
        android:layout_height="100dp"
        android:layout_marginTop="10dp"
        android:textSize="16sp"
        android:hint="Password"
        android:maxLines="1"
        android:inputType="textPassword"
        />
    <Button
        android:id="@+id/but0e"
        android:layout_width="400dp"
        android:layout_height="100dp"
        android:text="Login"
        android:background="@drawable/rounded_button_color_primary"
        />




</LinearLayout>
Main6：
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"

    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="10dp"
    android:gravity="center_horizontal"
    tools:context=".Main6Activity"
    android:background="@drawable/main1"
    >


    <TextView
        android:id="@+id/tv_func_ceshie"
        android:layout_width="300dp"
        android:layout_height="60dp"
        android:text="Please Select A Testing Project"
        android:layout_marginTop="30dp"
        android:textSize="25sp"
        android:gravity="center"
        android:background="@drawable/rounded_button_little_green"
        />

    <Button
        android:id="@+id/btn_duoji1e"
        android:layout_width="300dp"
        android:layout_height="100dp"
        android:text="Servo 1 Rotation Test"
        android:layout_marginTop="60dp"
        android:textSize="25sp"
        android:layout_gravity="center"
        android:textAllCaps="false"
        android:background="@drawable/rounded_button_little_blue"
        />
    <Button
        android:id="@+id/btn_duoji2e"
        android:layout_width="300dp"
        android:layout_height="100dp"
        android:text="Servo 2 Rotation Test"
        android:textSize="25sp"
        android:layout_marginTop="10dp"
        android:layout_gravity="center"
        android:textAllCaps="false"
        android:background="@drawable/rounded_button_little_blue"
        />
    <Button
        android:id="@+id/btn_colore"
        android:layout_width="300dp"
        android:layout_height="100dp"
        android:text=" Sensor Testing"
        android:layout_marginTop="10dp"
        android:textSize="25sp"
        android:layout_gravity="center"
        android:textAllCaps="false"
        android:background="@drawable/rounded_button_little_blue"
        />
    <!--<Button
        android:id="@+id/btn_margine"
        android:layout_width="300dp"
        android:layout_height="100dp"
        android:text="Margin Sensor Testing"
        android:textSize="25sp"
        android:layout_marginTop="10dp"
        android:layout_gravity="center"
        android:textAllCaps="false"
        android:background="@drawable/rounded_button_little_blue"
        />-->
    <com.friendlyarm.Utils.BorderScrollView
        android:id="@+id/scroolViewe"
        android:layout_width="match_parent"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:background="@color/colorPrimary">

        <TextView
            android:id="@+id/fromTextViewe"
            android:layout_width="match_parent"
            android:layout_height="fill_parent"
            android:background="@color/colorPrimary"
            android:gravity="top"
            android:singleLine="false"
            android:textColor="#000000" />
    </com.friendlyarm.Utils.BorderScrollView>

</LinearLayout>
Main7：
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_horizontal"
    android:orientation="vertical"
    android:padding="10dp"
    tools:context=".Main7Activity"
    android:background="@drawable/main1"
    >


    <Button
        android:id="@+id/sendButton10"
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:layout_marginTop="80dp"
        android:background="@drawable/rounded_button_little_green"
        android:text="欢迎来到智能药箱用户模式"
        android:textSize="25sp" />
   <!-- <Button
        android:id="@+id/sendButton3"
        android:layout_width="400dp"
        android:layout_height="90dp"
        android:layout_marginTop="160dp"
        android:background="@drawable/rounded_button_blue"
        android:text="用户模式"
        android:textSize="20sp" />-->

    <!-- <Button
         android:id="@+id/sendButton1"
         android:layout_width="350dp"
         android:layout_height="90dp"
         android:layout_marginTop="10dp"
         android:background="@drawable/rounded_button_yellow"
         android:text="黄色药品"
         android:textSize="20sp" />

     <Button
         android:id="@+id/sendButton2"
         android:layout_width="350dp"
         android:layout_height="90dp"
         android:layout_marginTop="10dp"
         android:background="@drawable/rounded_button_green"
         android:text="绿色药品"
         android:textSize="20sp" />

     <Button
         android:id="@+id/sendButton3"
         android:layout_width="350dp"
         android:layout_height="90dp"
         android:layout_marginTop="10dp"
         android:background="@drawable/rounded_button_blue"
         android:text="蓝色药品"
         android:textSize="20sp" />-->

    <Button
        android:id="@+id/sendButton20"
        android:layout_width="400dp"
        android:layout_height="90dp"
        android:layout_marginTop="100dp"
        android:background="@drawable/rounded_button_blue"
        android:text="点击开始运行"
        android:textSize="20sp" />


    <TextView
        android:id="@+id/label1"
        android:layout_width="400dp"
        android:layout_height="80dp"
        android:text="药品类型"
        android:gravity="center"
        android:textSize="25sp" />

    <com.friendlyarm.Utils.BorderScrollView
        android:id="@+id/scroolView"
        android:layout_width="match_parent"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:background="@color/colorPrimary">

        <TextView
            android:id="@+id/fromTextView"
            android:layout_width="match_parent"
            android:layout_height="fill_parent"
            android:background="@color/colorPrimary"
            android:gravity="top"
            android:singleLine="false"
            android:textColor="#000000" />
    </com.friendlyarm.Utils.BorderScrollView>

</LinearLayout>
Main8：
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_horizontal"
    android:orientation="vertical"
    android:padding="10dp"
    tools:context=".Main8Activity"
    android:background="@drawable/main1"
    >


    <Button
        android:id="@+id/sendButton10e"
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:layout_marginTop="80dp"
        android:background="@drawable/rounded_button_little_green"
        android:text="Welcome To User Mode"
        android:textAllCaps="false"
        android:textSize="25sp" />
    <!-- <Button
         android:id="@+id/sendButton3"
         android:layout_width="400dp"
         android:layout_height="90dp"
         android:layout_marginTop="160dp"
         android:background="@drawable/rounded_button_blue"
         android:text="用户模式"
         android:textSize="20sp" />-->

    <!-- <Button
         android:id="@+id/sendButton1"
         android:layout_width="350dp"
         android:layout_height="90dp"
         android:layout_marginTop="10dp"
         android:background="@drawable/rounded_button_yellow"
         android:text="黄色药品"
         android:textSize="20sp" />

     <Button
         android:id="@+id/sendButton2"
         android:layout_width="350dp"
         android:layout_height="90dp"
         android:layout_marginTop="10dp"
         android:background="@drawable/rounded_button_green"
         android:text="绿色药品"
         android:textSize="20sp" />

     <Button
         android:id="@+id/sendButton3"
         android:layout_width="350dp"
         android:layout_height="90dp"
         android:layout_marginTop="10dp"
         android:background="@drawable/rounded_button_blue"
         android:text="蓝色药品"
         android:textSize="20sp" />-->

    <Button
        android:id="@+id/sendButton20e"
        android:layout_width="400dp"
        android:layout_height="90dp"
        android:layout_marginTop="100dp"
        android:background="@drawable/rounded_button_blue"
        android:text="Click to start running"
        android:textAllCaps="false"
        android:textSize="20sp" />


    <TextView
        android:id="@+id/label1e"
        android:layout_width="400dp"
        android:layout_height="80dp"
        android:text="The Type Of Medicine"
        android:gravity="center"
        android:textSize="25sp" />

    <com.friendlyarm.Utils.BorderScrollView
        android:id="@+id/scroolViewe"
        android:layout_width="match_parent"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:background="@color/colorPrimary">

        <TextView
            android:id="@+id/fromTextViewe"
            android:layout_width="match_parent"
            android:layout_height="fill_parent"
            android:background="@color/colorPrimary"
            android:gravity="top"
            android:singleLine="false"
            android:textColor="#000000" />
    </com.friendlyarm.Utils.BorderScrollView>

</LinearLayout>
