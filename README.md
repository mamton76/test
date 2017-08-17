package com.yandex.suggest.network;

import android.os.Handler;
import android.os.HandlerThread;
import android.os.Looper;
import android.os.Message;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.robolectric.RobolectricTestRunner;
import org.robolectric.annotation.Config;

@RunWith(RobolectricTestRunner.class)
@Config(manifest = Config.NONE)
public class HandlerThreadTest {
    static final Object o = new Object();

    @Test //(timeout = 5000)
    public void eventsInHandlerThread() throws InterruptedException {
        MyHandlerThread thread = new MyHandlerThread();
        thread.start();
        Thread.sleep(1000);
        while (!thread.mReady) {
            Thread.sleep(100);
        }
        MyHandler mHandler = new MyHandler(thread.getLooper());
        mHandler.logAndSendEmptyMessage(1, 0);
        thread.mHandler.logAndSendEmptyMessage(2, 0);
        sleep();
    }

    @Test
    public void mainThread() throws InterruptedException {
        final MyHandler handler = new MyHandler(Looper.getMainLooper());
        Thread thread = new Thread() {
            @Override
            public void run() {
                handler.logAndSendEmptyMessage(2, 0);
            }
        };
        thread.start();
        handler.logAndSendEmptyMessage(1, 0);

        sleep();
    }

    @Test
    public void mainThreadFromOther() throws InterruptedException {
        final CreateMainHandlerThread thread = new CreateMainHandlerThread();
        thread.start();
        while (thread.mHandler == null) {
            Thread.sleep(500);
        }
        Thread thread1 = new Thread() {
            @Override
            public void run() {
                thread.mHandler.logAndSendEmptyMessage(2, 0);
            }
        };
        thread1.start();
        thread.mHandler.logAndSendEmptyMessage(1, 0);

        sleep();
    }

    @Test
    public void handlerThreadFromOther() throws InterruptedException {
        final MyHandlerThread thread = new MyHandlerThread();
        thread.start();
        while (thread.mHandler == null) {
            Thread.sleep(500);
        }
        Thread thread1 = new Thread() {
            @Override
            public void run() {
                thread.mHandler.logAndSendEmptyMessage(2, 0);
            }
        };
        thread1.start();
        thread.mHandler.logAndSendEmptyMessage(1, 0);

        sleep();
    }

    public void sleep() throws InterruptedException {
        synchronized (o) {
            System.out.println(System.currentTimeMillis() + " sleep " + Thread.currentThread().getName());
        }
        Thread.sleep(5000);
        synchronized (o) {
            System.out.println(System.currentTimeMillis() + " end " + Thread.currentThread().getName());
        }
    }

    private static class MyHandlerThread extends HandlerThread {
        public boolean mReady;
        public MyHandler mHandler;

        public MyHandlerThread() {
            super("HELLO");
        }

        @Override
        protected void onLooperPrepared() {
            super.onLooperPrepared();
            mHandler = new MyHandler(getLooper());
            mHandler.logAndSendEmptyMessage(3, 0);
            mHandler.logAndSendEmptyMessage(4, 2000);
            mReady = true;
        }

        @Override
        public void run() {
            super.run();
        }
    }

    private static class MyHandler extends Handler {
        public MyHandler() {
            super();
        }

        public MyHandler(Looper looper) {
            super(looper);
        }

        public void logAndSendEmptyMessage(int what, int delayMillis) {
            synchronized (o) {
                System.out.println(System.currentTimeMillis() + " " + this.hashCode() + " " + what + " send thread " + Thread.currentThread().getName());
            }
            sendEmptyMessageDelayed(what, delayMillis);
        }

        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            synchronized (o) {
                System.out.println(System.currentTimeMillis() + " " + this.hashCode() + " " + msg.what + " handle thread " + Thread.currentThread().getName());
            }
        }
    }

    private static class CreateMainHandlerThread extends Thread {
        public MyHandler mHandler;

        public CreateMainHandlerThread() {
            super("HELLO");
        }

        @Override
        public void run() {
            super.run();
            mHandler = new MyHandler(Looper.getMainLooper());
            mHandler.logAndSendEmptyMessage(3, 0);
        }
    }
}
