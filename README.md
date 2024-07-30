# async-job
 异步化执行封装
```php
/**
 * 异步化执行函数
 */
if (!function_exists('async_job')) {
    function async_job($callback) {
        dispatch(new AsyncJob(serialize(new SerializableClosure($callback))));
    }
}
```

执行demo：
```php
async_job(function () use($input) {
                    exec("sudo timedatectl set-ntp no");
                    foreach ($input['ntp_hosts'] as $host) {
                        $result = exec("sudo ntpdate $host");
                        if ($result) {
                            break;
                        }
                    }
                    exec("sudo hwclock --systohc");
                });
```


job：
```php
<?php
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

/**
 * hbl
 * 简单不复杂通用的job类，不用写多个job
 * @date 2023/09/19
 * Class AsyncJob
 * @package App\Jobs
 */
class AsyncJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public $timeout = 30000; //最大超时时间
    private $callback;


    public function __construct($callback)
    {
        $this->callback = unserialize($callback);
    }

    /**
     * 执行视频转码任务
     * 把非mp4的视频，转成mp4
     *
     * @return void
     */
    public function handle()
    {
        logger('----------start-async-job---------------------');
        is_callable($this->callback) && call_user_func($this->callback);
        logger('----------end-async-job---------------------');
    }
}

```
