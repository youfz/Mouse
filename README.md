# Mouse
android 实现虚拟鼠标方法

uinput（User Input）是一个用于模拟用户输入的子系统。uinput 允许开发者通过软件方式模拟硬件输入设备的事件，比如键盘按键、鼠标移动、触摸屏触摸等。这对于自动化测试、远程控制和模拟用户交互等场景非常有用

具体步骤：
       1、打开uinput设备，获取设备文件描述符
          open("/dev/uinput", O_WRONLY | O_NONBLOCK)
       2、设置 uinput 设备参数，创建鼠标： 使用 ioctl 系统调用设置 uinput 设备的参数，如设备名称、支持的事件类型等
          //config uinput working mode,  mouse or touchscreen?  relative coordinates or absolute coordinate?
          if (ioctl(mouse_fd, UI_SET_EVBIT, EV_KEY) < 0)         //support key button
              LOGD("error: ioctl");
          if (ioctl(mouse_fd, UI_SET_KEYBIT, BTN_LEFT) < 0)  //support mouse left key
              LOGD("error: ioctl");
      
          if (ioctl(mouse_fd, UI_SET_KEYBIT, BTN_RIGHT) < 0)  //support mouse right key
              LOGD("error: ioctl");
      
          if (ioctl(mouse_fd, UI_SET_EVBIT, EV_REL) < 0)       //uinput use relative coordinates
              LOGD("error: ioctl");
          if (ioctl(mouse_fd, UI_SET_RELBIT, REL_X) < 0)         //uinput use x coordinates
              LOGD("error: ioctl");
          if (ioctl(mouse_fd, UI_SET_RELBIT, REL_Y) < 0)         //uinput use y coordinates
              LOGD("error: ioctl");

          memset(&u_input_dev, 0,
                 sizeof(u_input_dev));                  //creat an virtul input device node in /dev/input/***
          snprintf(u_input_dev.name, UINPUT_MAX_NAME_SIZE, "uinput-sample");
          u_input_dev.id.bustype = BUS_USB;
          u_input_dev.id.vendor = 0x1;
          u_input_dev.id.product = 0x1;
          u_input_dev.id.version = 1;
      
          if (write(mouse_fd, &u_input_dev, sizeof(u_input_dev)) < 0)
              LOGD("error: write");
      
          if (ioctl(mouse_fd, UI_DEV_CREATE) < 0)
              LOGD("error: ioctl");
      3、创建鼠标事件结构体
          memset(&inputEvent, 0, sizeof(struct input_event));
          inputEvent.type = EV_REL;
          inputEvent.code = REL_X;
          inputEvent.value = mouse_dx;
          LOGD("move mouse  02\n");
          if (write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0) {
              LOGD("move error\n");
          }
    4、发送鼠标事件
         write(uinput_fd, &ev, sizeof(ev));

    5、关闭 uinput 设备
         close(uinput_fd);


普通事件处理
    1、移动
    void Mouse::move(int direction,int x,int y) {
        LOGD("move mouse  01 mouse_dx direction = %d ,x = %d ,y = %d",direction,x,y);
        mouse_dx = x;
        mouse_dy = y;
        LOGD("mouse_dx x = %d,mouse_dy = %d.", mouse_dx, mouse_dy);
        memset(&inputEvent, 0, sizeof(struct input_event));
        inputEvent.type = EV_REL;
        inputEvent.code = REL_X;
        inputEvent.value = mouse_dx;
        LOGD("move mouse  02\n");
        if (write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0) {
            LOGD("move error\n");
        }
        LOGD("move mouse  03\n");
        memset(&inputEvent, 0, sizeof(struct input_event));
        inputEvent.type = EV_REL;
        inputEvent.code = REL_Y;
        inputEvent.value = mouse_dy;
        LOGD("move mouse  04\n");
        if (write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0) {
            LOGD("move error\n");
        }
        LOGD("move mouse  05\n");
        memset(&inputEvent, 0, sizeof(struct input_event));
        inputEvent.type = EV_SYN;
        inputEvent.code = SYN_REPORT;
        inputEvent.value = 0;
        if (write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0) {
            LOGD("move error\n");
        }
        LOGD("move mouse  06\n");
    }

    2、鼠标事件  单击、右击、滚轮事件
        void Mouse::report_key(int type, int keycode, int value) {
             void Mouse::report_key(int type, int keycode, int value) {

    if(type == 1){

        for(i = 0; i < 2; i++){
            memset(&inputEvent, 0, sizeof(struct input_event));
            inputEvent.type = EV_KEY;  //mouse left key
            inputEvent.code = BTN_LEFT;
            inputEvent.value = 1;
            if(write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0)
                LOGD("error: write");

            memset(&inputEvent, 0, sizeof(struct input_event));
            inputEvent.type = EV_SYN; // inform input system to process this input event
            inputEvent.code = 0;
            inputEvent.value = 0;
            if(write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0)
                LOGD("error: write");
            usleep(15000);
        }

        usleep(15000);

        for(i = 0; i < 3; i++){
            memset(&inputEvent, 0, sizeof(struct input_event));
            inputEvent.type = EV_KEY;  //mouse left key
            inputEvent.code = BTN_LEFT;
            inputEvent.value = 0;
            if(write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0)
                LOGD("error: write");

            memset(&inputEvent, 0, sizeof(struct input_event));
            inputEvent.type = EV_SYN; // inform input system to process this input event
            inputEvent.code = 0;
            inputEvent.value = 0;
            if(write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0)
                LOGD("error: write");
            usleep(15000);
        }
    }else if (type == 2) {
        //模拟鼠标右键点击
        //mouse click right key begin
        for (i = 0; i < 3; i++) {
            memset(&inputEvent, 0, sizeof(struct input_event));
            inputEvent.type = EV_KEY;  //mouse right key
            inputEvent.code = BTN_RIGHT;
            inputEvent.value = 1;
            if (write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0)
                LOGD("error: write");

            memset(&inputEvent, 0, sizeof(struct input_event));
            inputEvent.type = EV_SYN; // inform input system to process this input event
            inputEvent.code = 0;
            inputEvent.value = 0;
            if (write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0)
                LOGD("error: write");
            usleep(15000);
        }

        usleep(15000);

        for (i = 0; i < 3; i++) {
            memset(&inputEvent, 0, sizeof(struct input_event));
            inputEvent.type = EV_KEY;  //mouse right key
            inputEvent.code = BTN_RIGHT;
            inputEvent.value = 0;
            if (write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0)
                LOGD("error: write");

            memset(&inputEvent, 0, sizeof(struct input_event));
            inputEvent.type = EV_SYN; // inform input system to process this input event
            inputEvent.code = 0;
            inputEvent.value = 0;
            if (write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0)
                LOGD("error: write");
            usleep(15000);
         }
      }else if (type == 3) {
        // 模拟滚轮事件
//        for (int j = 0; j < 1500; j++) {
        LOGD("模拟滚轮事件 type = %d,keycode = %d", type, keycode);
        memset(&inputEvent, 0, sizeof(struct input_event));

        inputEvent.type = EV_REL;
        inputEvent.code = REL_WHEEL;
        if (keycode >= 0) {
            inputEvent.value = 1;  // 正向滚动
        } else {
            inputEvent.value = -1;  // 反向滚动
        }
        if (write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0)
            LOGD("error: write");

        memset(&inputEvent, 0, sizeof(struct input_event));
        inputEvent.type = EV_SYN; // inform input system to process this input event
        inputEvent.code = 0;
        inputEvent.value = 0;
        if (write(mouse_fd, &inputEvent, sizeof(struct input_event)) < 0)
            LOGD("error: write");
        usleep(15000);
    }
