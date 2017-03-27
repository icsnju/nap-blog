title: Spring Transaction Manager学习笔记
category: TVDCR
date: 2015-11-10
tags: [spring, transacion, dynamic reconfiguration]
author: Hooting
---
使用Spring声明式事务处理，是通过结合IoC容器和Spring已有的TransactionProxyFactoryBean来对事务管理进行配置。
在TransactionProxyFactoryBean中为事务方法配置传播行为、并发事务隔离级别这些事务处理的属性，
从而对声明式事务的处理提供指导。
声明式事务处理的实现，大致可以分为以下三个部分：
* 读取和处理在IoC容器中配置的事务处理属性， 并转为Spring事务处理需要的内部数据结构。
* Spring事务处理模块实现的统一的事务处理过程。包含了处理事务配置属性，以及与现场绑定完成事务处理。Spring通过TransactionInfo和TransactionStatus两个数据对象，在事务处理过程中记录和传递相关执行场景。
* 对于底层的事务操作，Spring委托给具体的事务处理器来完成，即PlatformTransactionManager的具体实现。比如DataSourceTransactionManager和HibernateTransactionManager。

<!--more-->

---

## 声明式事务处理的基本过程
使用Spring声明式事务处理，是通过结合IoC容器和Spring已有的TransactionProxyFactoryBean来对事务管理进行配置。
在TransactionProxyFactoryBean中为事务方法配置传播行为、并发事务隔离级别这些事务处理的属性，
从而对声明式事务的处理提供指导。
声明式事务处理的实现，大致可以分为以下三个部分：
* 读取和处理在IoC容器中配置的事务处理属性， 并转为Spring事务处理需要的内部数据结构。
* Spring事务处理模块实现的统一的事务处理过程。包含了处理事务配置属性，以及与现场绑定完成事务处理。Spring通过TransactionInfo和TransactionStatus两个数据对象，在事务处理过程中记录和传递相关执行场景。
* 对于底层的事务操作，Spring委托给具体的事务处理器来完成，即PlatformTransactionManager的具体实现。比如DataSourceTransactionManager和HibernateTransactionManager。

## 事务处理拦截器的配置
配置工作，包括设置拦截器TransactionInterceptor、通知器DefaultPointcutAdvisor、
注入进来的PlatformTransactionManager和事务处理属性TransactionAttribute。
这些工作是由IoC的TransactionProxyFactoryBean完成的。它的实现如下：

```
public class TransactionProxyFactoryBean extends AbstractSingletonProxyFactoryBean
		implements BeanFactoryAware {
	/**
	 * 这个拦截器通过AOP发挥作用，通过这个拦截器，Spring封装了事务处理实现
	 */
	private final TransactionInterceptor transactionInterceptor = new TransactionInterceptor();

	private Pointcut pointcut;


	/**
	 * Set the default transaction manager. This will perform actual
	 * transaction management: This class is just a way of invoking it.
	 * @see TransactionInterceptor#setTransactionManager
	 */
	public void setTransactionManager(PlatformTransactionManager transactionManager) {
		this.transactionInterceptor.setTransactionManager(transactionManager);
	}

	/**
	 * Creates an advisor for this FactoryBean's TransactionInterceptor.
	 * 创建Spring AOP对事务处理的AOP
	 */
	@Override
	protected Object createMainInterceptor() {
		this.transactionInterceptor.afterPropertiesSet();
		if (this.pointcut != null) {
			return new DefaultPointcutAdvisor(this.pointcut, this.transactionInterceptor);
		}
		else {
			// Rely on default pointcut.
			return new TransactionAttributeSourceAdvisor(this.transactionInterceptor);
		}
	}

}
```

有上面的代码可以知道，TransactionInterceptor在方法`createMainInterceptor`中被配置为Advisor通知器的一部分。
而`createMainInterceptor`方法在IoC容器完成Bean的注入依赖时，通过`initiaizeBean`方法被调用，调用过程如下图:
![](/images/createMainInterceptor.png)

这个`afterPropertiesSet`方法的功能实现如下所示。从代码中可以看到，在建立TransactionProxyFactoryBean的事务
处理拦截器的时候，首先需要对ProxyFactoryBean的目标Bean设置进行检查，如果这个目标Bean的配置是正确的，
就会通过创建一个ProxyFactory对象，从而实现AOP的使用。

```
public void afterPropertiesSet() {
    if (this.target == null) {
    	throw new IllegalArgumentException("Property 'target' is required");
    }
    if (this.target instanceof String) {
    	throw new IllegalArgumentException("'target' needs to be a bean reference, not a bean name as value");
    }
    if (this.proxyClassLoader == null) {
    	this.proxyClassLoader = ClassUtils.getDefaultClassLoader();
    }
    //TransactionFactoryBean使用ProxyFactory完成AOP的基本功能
    ProxyFactory proxyFactory = new ProxyFactory();
    
    if (this.preInterceptors != null) {
    	for (Object interceptor : this.preInterceptors) {
    		proxyFactory.addAdvisor(this.advisorAdapterRegistry.wrap(interceptor));
    	}
    }
    
    // Add the main interceptor (typically an Advisor).
    proxyFactory.addAdvisor(this.advisorAdapterRegistry.wrap(createMainInterceptor()));
    
    if (this.postInterceptors != null) {
    	for (Object interceptor : this.postInterceptors) {
    		proxyFactory.addAdvisor(this.advisorAdapterRegistry.wrap(interceptor));
    	}
    }
    
    proxyFactory.copyFrom(this);
    //设置目标源
    TargetSource targetSource = createTargetSource(this.target);
    proxyFactory.setTargetSource(targetSource);
    
    if (this.proxyInterfaces != null) {
    	proxyFactory.setInterfaces(this.proxyInterfaces);
    }
    else if (!isProxyTargetClass()) {
    	// Rely on AOP infrastructure to tell us what interfaces to proxy.
    	proxyFactory.setInterfaces(
    			ClassUtils.getAllInterfacesForClass(targetSource.getTargetClass(), this.proxyClassLoader));
    }
    
    this.proxy = proxyFactory.getProxy(this.proxyClassLoader);
}
```

## 事务处理拦截器的实现
对事务方法的拦截是通过`invoke`方法，它使Proxy代理对象的回调方法。在事务处理拦截器TransactionInterceptor中，
`invoke`方法的实现代码如下。可以看到，首先获得调用方法的事务处理配置，之后取得配置的PlatformTransactionManager
,由这个事务处理器来实现事务的创建、提交、回滚操作。

```
public Object invoke(final MethodInvocation invocation) throws Throwable {
	// Work out the target class: may be {@code null}.
	// The TransactionAttributeSource should be passed the target class
	// as well as the method, which may be from an interface.
	Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);

	// Adapt to TransactionAspectSupport's invokeWithinTransaction...
	return invokeWithinTransaction(invocation.getMethod(), targetClass, new InvocationCallback() {
		@Override
		public Object proceedWithInvocation() throws Throwable {
			return invocation.proceed();
		}
	});
}


protected Object invokeWithinTransaction(Method method, Class<?> targetClass, final InvocationCallback invocation)
		throws Throwable {

	// If the transaction attribute is null, the method is non-transactional.
	final TransactionAttribute txAttr = getTransactionAttributeSource().getTransactionAttribute(method, targetClass);
	final PlatformTransactionManager tm = determineTransactionManager(txAttr);
	final String joinpointIdentification = methodIdentification(method, targetClass);
	/**
	 * 区分不同类型的PlatformTransactionManager, 因为他们的调用方式不同，
	 * 像DataSourceTransactionManager来说，不是CallbackPreferringPlatformTransactionManager,
	 * 不需要通过回调的方式来使用。
	 */
	if (txAttr == null || !(tm instanceof CallbackPreferringPlatformTransactionManager)) {
		// Standard transaction demarcation with getTransaction and commit/rollback calls.
		//TransactionInfo是保存当前事务状态的对象
		TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
		Object retVal = null;
		try {
			// This is an around advice: Invoke the next interceptor in the chain.
			// This will normally result in a target object being invoked.
			retVal = invocation.proceedWithInvocation();
		}
		catch (Throwable ex) {
			// target invocation exception
			completeTransactionAfterThrowing(txInfo, ex);
			throw ex;
		}
		finally {
			cleanupTransactionInfo(txInfo);
		}
		commitTransactionAfterReturning(txInfo);
		return retVal;
	}

	else {
		.....
	}
}
```


##事务的创建
我们注意到TransactionInterceptor拦截器的`invoke`回调中使用的`createTransactionIfNecessary
`方法，它是事务创建的起点。它的实现代码如下。在`createTransactionIfNecessary
`方法调用中，可以看到两个重要的数据对象TransactionStatus和TransactionInfo的调用，这两个对象持有的
数据是事务处理器对事务进行处理的主要依据，对他们的使用贯穿着整个事务处理的过程。

```
protected TransactionInfo createTransactionIfNecessary(
		PlatformTransactionManager tm, TransactionAttribute txAttr, final String joinpointIdentification) {

	// If no name specified, apply method identification as transaction name.
	if (txAttr != null && txAttr.getName() == null) {
		txAttr = new DelegatingTransactionAttribute(txAttr) {
			@Override
			public String getName() {
				return joinpointIdentification;
			}
		};
	}

	TransactionStatus status = null;
	if (txAttr != null) {
		if (tm != null) {
			/**
			 * 这里使用我们定义好的事务方法的配置信息。
			 * 事务创建由事务处理器来完成，同时返回TransactionStatus来记录
			 * 当前的事务状态，包括已经创建的事务。
			 */
			status = tm.getTransaction(txAttr);
		}
		else {
			if (logger.isDebugEnabled()) {
				logger.debug("Skipping transactional joinpoint [" + joinpointIdentification +
						"] because no transaction manager has been configured");
			}
		}
	}
	//准备TransactionInfo，它封装了事务处理的配置信息以及TransactionStatus
	return prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);
}


protected TransactionInfo prepareTransactionInfo(PlatformTransactionManager tm,
		TransactionAttribute txAttr, String joinpointIdentification, TransactionStatus status) {

	TransactionInfo txInfo = new TransactionInfo(tm, txAttr, joinpointIdentification);
	if (txAttr != null) {
		// We need a transaction for this method
		if (logger.isTraceEnabled()) {
			logger.trace("Getting transaction for [" + txInfo.getJoinpointIdentification() + "]");
		}
		// The transaction manager will flag an error if an incompatible tx already exists
		txInfo.newTransactionStatus(status);
	}
	else {
		// The TransactionInfo.hasTransaction() method will return
		// false. We created it only to preserve the integrity of
		// the ThreadLocal stack maintained in this class.
		if (logger.isTraceEnabled())
			logger.trace("Don't need to create transaction for [" + joinpointIdentification +
					"]: This method isn't transactional.");
	}

	// We always bind the TransactionInfo to the thread, even if we didn't create
	// a new transaction here. This guarantees that the TransactionInfo stack
	// will be managed correctly even if no transaction was created by this aspect.
	txInfo.bindToThread();
	return txInfo;
}
```

在以上的处理过程完成以后，可以看到具体的事务创建就交给事务处理器来完成了。下面看事务处理器中去了解
一下更底层的事务创建过程，它被上述代码的`tm.getTransaction(txAttr)`调用出发，生成一个TransactionStatus对象，
封装了底层事务对象的创建。在AbstractPlatformTransactionManager中，提供了创建事务的模板，代码如下所示。
AbstractPlatformTransactionManager会根据事务属性配置和当前线程绑定的事务信息，对事务是否需要创建、怎样创建
进行一些通用的处理，然后把事务创建的底层工作交给具体的事务处理器完成(如DataSourceTransactionManagera)。

```
public final TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException {
	/**
	 * 这个doGetTransaction是抽象函数，Transaction对象的取得由具体的事务处理器实现。
	 * 比如DataSourceTransactionManager
	 */
	Object transaction = doGetTransaction();

	// Cache debug flag to avoid repeated checks.
	boolean debugEnabled = logger.isDebugEnabled();

	if (definition == null) {
		/*
		 * 使用默认的DefaultTransactionDefinition：
		 * propagationBehavior=PROPAGATION_REQUIRED
		 * isolationLevel=ISOLATION_DEFAULT;
		 * timeout=TIMEOUT_DEFAULT;
		 * readOnly=false;
		 */
		// Use defaults if no transaction definition given.
		definition = new DefaultTransactionDefinition();
	}
	//如果当前线程存在事务，需要根据事务传播属性生成事务
	if (isExistingTransaction(transaction)) {
		// Existing transaction found -> check propagation behavior to find out how to behave.
		return handleExistingTransaction(definition, transaction, debugEnabled);
	}

	// Check definition settings for new transaction.
	if (definition.getTimeout() < TransactionDefinition.TIMEOUT_DEFAULT) {
		throw new InvalidTimeoutException("Invalid transaction timeout", definition.getTimeout());
	}

	// No existing transaction found -> check propagation behavior to find out how to proceed.
	if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_MANDATORY) {
		throw new IllegalTransactionStateException(
				"No existing transaction found for transaction marked with propagation 'mandatory'");
	}
	else if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRED ||
			definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRES_NEW ||
			definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NESTED) {
		SuspendedResourcesHolder suspendedResources = suspend(null);
		if (debugEnabled) {
			logger.debug("Creating new transaction with name [" + definition.getName() + "]: " + definition);
		}
		try {
			boolean newSynchronization = (getTransactionSynchronization() != SYNCHRONIZATION_NEVER);
			DefaultTransactionStatus status = newTransactionStatus(
					definition, transaction, true, newSynchronization, debugEnabled, suspendedResources);
			/*
			 * 这里是创建事务的调用，由具体的事务处理器完成，比如：
			 * HibernateTransactionManager和DataSourceTransactionManager
			 */
			doBegin(transaction, definition);
			prepareSynchronization(status, definition);
			return status;
		}
		catch (RuntimeException ex) {
			resume(null, suspendedResources);
			throw ex;
		}
		catch (Error err) {
			resume(null, suspendedResources);
			throw err;
		}
	}
	else {
		// Create "empty" transaction: no actual transaction, but potentially synchronization.
		if (definition.getIsolationLevel() != TransactionDefinition.ISOLATION_DEFAULT && logger.isWarnEnabled()) {
			logger.warn("Custom isolation level specified but no actual transaction initiated; " +
					"isolation level will effectively be ignored: " + definition);
		}
		boolean newSynchronization = (getTransactionSynchronization() == SYNCHRONIZATION_ALWAYS);
		return prepareTransactionStatus(definition, null, true, newSynchronization, debugEnabled, null);
	}
}

/**
 * Create a TransactionStatus for an existing transaction.
 */
private TransactionStatus handleExistingTransaction(
		TransactionDefinition definition, Object transaction, boolean debugEnabled)
		throws TransactionException {

	if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NEVER) {
		throw new IllegalTransactionStateException(
				"Existing transaction found for transaction marked with propagation 'never'");
	}

	if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NOT_SUPPORTED) {
		if (debugEnabled) {
			logger.debug("Suspending current transaction");
		}
		Object suspendedResources = suspend(transaction);
		boolean newSynchronization = (getTransactionSynchronization() == SYNCHRONIZATION_ALWAYS);
		return prepareTransactionStatus(
				definition, null, false, newSynchronization, debugEnabled, suspendedResources);
	}

	if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRES_NEW) {
		if (debugEnabled) {
			logger.debug("Suspending current transaction, creating new transaction with name [" +
					definition.getName() + "]");
		}
		SuspendedResourcesHolder suspendedResources = suspend(transaction);
		try {
			boolean newSynchronization = (getTransactionSynchronization() != SYNCHRONIZATION_NEVER);
			DefaultTransactionStatus status = newTransactionStatus(
					definition, transaction, true, newSynchronization, debugEnabled, suspendedResources);
			doBegin(transaction, definition);
			prepareSynchronization(status, definition);
			return status;
		}
		catch (RuntimeException beginEx) {
			resumeAfterBeginException(transaction, suspendedResources, beginEx);
			throw beginEx;
		}
		catch (Error beginErr) {
			resumeAfterBeginException(transaction, suspendedResources, beginErr);
			throw beginErr;
		}
	}

	if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NESTED) {
		if (!isNestedTransactionAllowed()) {
			throw new NestedTransactionNotSupportedException(
					"Transaction manager does not allow nested transactions by default - " +
					"specify 'nestedTransactionAllowed' property with value 'true'");
		}
		if (debugEnabled) {
			logger.debug("Creating nested transaction with name [" + definition.getName() + "]");
		}
		if (useSavepointForNestedTransaction()) {
			// Create savepoint within existing Spring-managed transaction,
			// through the SavepointManager API implemented by TransactionStatus.
			// Usually uses JDBC 3.0 savepoints. Never activates Spring synchronization.
			DefaultTransactionStatus status =
					prepareTransactionStatus(definition, transaction, false, false, debugEnabled, null);
			status.createAndHoldSavepoint();
			return status;
		}
		else {
			// Nested transaction through nested begin and commit/rollback calls.
			// Usually only for JTA: Spring synchronization might get activated here
			// in case of a pre-existing JTA transaction.
			boolean newSynchronization = (getTransactionSynchronization() != SYNCHRONIZATION_NEVER);
			DefaultTransactionStatus status = newTransactionStatus(
					definition, transaction, true, newSynchronization, debugEnabled, null);
			doBegin(transaction, definition);
			prepareSynchronization(status, definition);
			return status;
		}
	}

	// Assumably PROPAGATION_SUPPORTS or PROPAGATION_REQUIRED.
	if (debugEnabled) {
		logger.debug("Participating in existing transaction");
	}
	if (isValidateExistingTransaction()) {
		if (definition.getIsolationLevel() != TransactionDefinition.ISOLATION_DEFAULT) {
			Integer currentIsolationLevel = TransactionSynchronizationManager.getCurrentTransactionIsolationLevel();
			if (currentIsolationLevel == null || currentIsolationLevel != definition.getIsolationLevel()) {
				Constants isoConstants = DefaultTransactionDefinition.constants;
				throw new IllegalTransactionStateException("Participating transaction with definition [" +
						definition + "] specifies isolation level which is incompatible with existing transaction: " +
						(currentIsolationLevel != null ?
								isoConstants.toCode(currentIsolationLevel, DefaultTransactionDefinition.PREFIX_ISOLATION) :
								"(unknown)"));
			}
		}
		if (!definition.isReadOnly()) {
			if (TransactionSynchronizationManager.isCurrentTransactionReadOnly()) {
				throw new IllegalTransactionStateException("Participating transaction with definition [" +
						definition + "] is not marked as read-only but existing transaction is");
			}
		}
	}
	boolean newSynchronization = (getTransactionSynchronization() != SYNCHRONIZATION_NEVER);
	return prepareTransactionStatus(definition, transaction, false, newSynchronization, debugEnabled, null);
}
```

## 具体事务处理器的实现
下面，以DataSourceTransactionManager为例，简单介绍一下在具体的事务处理器中，是如何实现事务的创建、提交、
回滚这些底层的事务处理操作。在实现过程中，需要把数据库的Connection和当前的线程进行绑定。对于事务的提交
和回滚，都是直接调用Connection的提交和回滚方法来完成。

```
public class DataSourceTransactionManager extends AbstractPlatformTransactionManager
		implements ResourceTransactionManager, InitializingBean {

	private DataSource dataSource;

	@Override
	protected Object doGetTransaction() {
		DataSourceTransactionObject txObject = new DataSourceTransactionObject();
		txObject.setSavepointAllowed(isNestedTransactionAllowed());
		ConnectionHolder conHolder =
				(ConnectionHolder) TransactionSynchronizationManager.getResource(this.dataSource);
		txObject.setConnectionHolder(conHolder, false);
		return txObject;
	}
```

TransactionSynchronizationManager保存了线程级别的变量，当处于同一个事务的方法调用doGetTransaction时，getResource方法返回的是同一个ConnectionHolder

```
public abstract class TransactionSynchronizationManager {

	private static final Log logger = LogFactory.getLog(TransactionSynchronizationManager.class);

	private static final ThreadLocal<Map<Object, Object>> resources =
			new NamedThreadLocal<Map<Object, Object>>("Transactional resources");
	/**
	 * Retrieve a resource for the given key that is bound to the current thread.
	 * @param key the key to check (usually the resource factory)
	 * @return a value bound to the current thread (usually the active
	 * resource object), or {@code null} if none
	 * @see ResourceTransactionManager#getResourceFactory()
	 */
	public static Object getResource(Object key) {
		Object actualKey = TransactionSynchronizationUtils.unwrapResourceIfNecessary(key);
		Object value = doGetResource(actualKey);
		if (value != null && logger.isTraceEnabled()) {
			logger.trace("Retrieved value [" + value + "] for key [" + actualKey + "] bound to thread [" +
					Thread.currentThread().getName() + "]");
		}
		return value;
	}

	/**
	 * Actually check the value of the resource that is bound for the given key.
	 */
	private static Object doGetResource(Object actualKey) {
		Map<Object, Object> map = resources.get();
		if (map == null) {
			return null;
		}
		Object value = map.get(actualKey);
		// Transparently remove ResourceHolder that was marked as void...
		if (value instanceof ResourceHolder && ((ResourceHolder) value).isVoid()) {
			map.remove(actualKey);
			// Remove entire ThreadLocal if empty...
			if (map.isEmpty()) {
				resources.remove();
			}
			value = null;
		}
		return value;
	}
}
```

