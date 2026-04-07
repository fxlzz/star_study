实现功能：
1. 支持手动触发和自动触发
2. 支持轮询
3. 支持超时重传
4. 支持 loading、error 状态管理
5. 支持竞态处理

```js
import { useState, useEffect, useRef, useCallback } from "react";

interface AsyncOptions<T, P extends any[]> {
	manual?: boolean;  // 是否手动触发
	params?: P; 	   // 自动触发时参数
	pollingInterval?: number; // 轮询间隔 ms
	timeout?: number;  // 超时时间 ms
	retryCount?: number; // 失败重试次数
	onSuccess?: (data: T) => void;
	onError?: (err: any) => void;
}

function useAsync<T, P extends any[]>(
	fetch: (...args: P) => Promise<T>,
	options: AsyncOptions<T, P> = {}
) {
	const {
		manual = false,
		params = [] as unknown as P,
		pollingInterval = 0,
		timeout = 5000,
		retryCount = 0,
		onSuccess,
		onError
	} = options;

	const [data, setData] = useState<T | null>(null);
	const [loading, setLoading] = useState<boolean>(!manual);
	const [error, setError] = useState<any>(null);
	
	const id = useRef<number>(0);
	const controller = useRef<AbortController | null>(null);
	const timer = useRef<NodeJS.Timeout | null>(null);
	const retryCount = useRef(0);
	
	const run = useCallback(async (...args: P): Promise<T | undefined>) => {
		// 取消上一个未完成的请求
		if(controller.current) {
			controller.current.abort();
		}	
		controller.current = new AbortController();
		
		// 处理竞态问题
		const currentTime = Date.now();
		id.current = currentTime;
		
		setLoading(true);
		setError(null);
		
		// 超时机制
		let timeoutTimer: NodeJS.Timeout;
		const timeoutPromise = new Promise((_, reject) => {
			if(timeout > 0) {
				timer = setTimeout(() => {
					controller.current?.abort();
					reject(new Error("请求超时"));
				}, timeout)
			}
		});
		
		try {
			// 合并请求与超时判断
			const result = await Promise.race([
				fetch(...args),
				timeoutPromise
			]) as T;
			
			// 竞态检查
			if (currentTime !== id.current) {
				return;
			}
			
			setData(result);
			retryCount.current = 0; // 成功后重置重试次数
			onSuccess?.(result);
			
			return result;
			
		} catch (err: any) {
			if(err.name === "AbortError") return;
			
			if(retryCount.current < retryCount) {
				retryCount.current++; 
				return run(...args); // 未到次数重传
			}
			
			if(currentTime === id.current) {
				setError(err);
				onError?.(err);
			}
		} finally {
			if(currentTime === id.current) {
				setLoading(false);
			}
			
			if(timer) clearTimeout(timer);
		}
	}, [fetch, timeout, retryCount, onSuccess, onError]);
	
	// 允许手动取消
	const cancel = useCallback(() => {
		controller.current?.abort();
		setLoading(false);
	}, []);
	
	useEffct(() => {
		// 自动触发，params参数改变时才触发
		if(!manual) {
			run(...params);
		}
		
		// 轮询
		if(pollingInterval > 0) {
			timer.current = setInterval(() => {
				run(...params);
			}, pollingInterval)
		}
		
		return () => {
			cancel();
			if(timer.current) clearInterval(timer.current);
		}
	}, [manual, JSON.stringify(params), pollingInterval, run, cancel]);
	
	return {
		data, 
		loading,
		error,
		run, 
		cancel
	}
}
```