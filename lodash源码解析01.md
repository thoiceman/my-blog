# 前言
因为工作中lodash用的比较多，本着刨根问底的学习精神，深入研究下源码，看看日常使用的方法是如何实现？通过研究，也会让日常使用更加得心应手。




# 其他
```js
  /**
   * A faster alternative to `Function#apply`, this function invokes `func`
   * with the `this` binding of `thisArg` and the arguments of `args`.
   *
   * @private
   * @param {Function} func The function to invoke.
   * @param {*} thisArg The `this` binding of `func`.
   * @param {Array} args The arguments to invoke `func` with.
   * @returns {*} Returns the result of `func`.
   */
  function apply(func, thisArg, args) {
    switch (args.length) {
      case 0: return func.call(thisArg);
      case 1: return func.call(thisArg, args[0]);
      case 2: return func.call(thisArg, args[0], args[1]);
      case 3: return func.call(thisArg, args[0], args[1], args[2]);
    }
    return func.apply(thisArg, args);
  }
 ``` 
 这个 apply 实现通过 参数数量分段优化，在参数较少时直接展开调用 call，参数较多时才使用 apply，从而在 Lodash 的高性能场景（如数组操作、迭代器）中显著提升执行效率。这是典型的大规模库针对高频操作的微优化策略。
* 为什么这样设计？
    * 在参数较少时，直接展开参数调用 call 避免了 apply 处理数组时的额外开销
    * 参数较多时，apply 的效率反而更高（因为手动展开参数会更繁琐）

```js
  /**
   * A specialized version of `_.forEach` for arrays without support for
   * iteratee shorthands.
   *
   * @private
   * @param {Array} [array] The array to iterate over.
   * @param {Function} iteratee The function invoked per iteration.
   * @returns {Array} Returns `array`.
   */
  function arrayEach(array, iteratee) {
    var index = -1,
        length = array == null ? 0 : array.length;

    while (++index < length) {
      if (iteratee(array[index], index, array) === false) {
        break;
      }
    }
    return array;
  }
```
该 arrayEach 函数是 Lodash 内部用于 数组遍历的专用实现，其核心作用是 高效执行数组元素的迭代操作，并支持通过返回 false 提前终止循环