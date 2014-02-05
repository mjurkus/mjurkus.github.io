---
title: "Primitive Android memory leak safeguard"
description: "We all had problems with memory leaks in our apps. And tracking them down is a painfull painfull tedious and painfull job"
layout: post
tags: memory-leak, android
disqus_comments: true
---

We all using fragments and using global variables for our views. But if you're at all like me - you will forget to destroy references to those views now and then. And that could lead to some problems when they start to leak memory.

Here is simple and quite primitive way to track those in your development environment.  
Add this to your `BaseFragment` and put your mind at ease.

{% highlight java %}  
@Override
public void onDestroy() {
    super.onDestroy();

    if (BuildConfig.DEBUG) {
        Class clazz = getClass();
        int leakCount = 0;

        while (MDFragment.class.isAssignableFrom(clazz)) {
            Field[] fields = clazz.getDeclaredFields();

            for (Field field : fields) {
                field.setAccessible(true);
                try {
                    Object o = field.get(this);

                    if (o != null && o instanceof View) {
                        Log.w(String.format("Potential memory leak at field '%s#%s'", clazz.getSimpleName(), field.getName()));
                        leakCount++;
                    }
                } catch (IllegalAccessException e) {
                    Log.e(e);
                }
            }
             
            clazz = clazz.getSuperclass();
        }

        if (leakCount > 0) {
            Toast.makeText(getActivity(),
                String.format("There are %d potential memory leaks in %s or its super class. See log for details.", leakCount, getClass().getSimpleName()), Toast.LENGTH_SHORT).show();
        }
    }
}
{% endhighlight %}
