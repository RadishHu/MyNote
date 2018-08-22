## BinarySearch

二分法用于从一堆数中找出指定的数，这一堆数需要符合的特征：

- 存储在数组中，如果用链表存储，无法使用二分法查找
- 有序排列

二分法实现：

- 循环实现

  ```java
  public static int rank(int key,int[] arr) {
  
      int start = 0;
      int end = arr.length -1;
  
      while(start <= end) {
          int mid = start + (end - start)/2;
          if (key < arr[mid]) end = mid -1;
          else if (key > arr[mid]) start = mid + 1;
          else return mid;
      }
      return -1;
  }
  ```

- 递归实现

  ```java
  public static int rank2(int key,int[] arr,int start,int end) {
  
      if (start > end) return -1;
  
      int mid = (end - start) / 2 + start;
  
      if (key < arr[mid]) return rank2(key,arr,start,mid -1);
      else if (key > arr[mid]) return rank2(key,arr,mid + 1,end);
      else return mid;
  }
  
  public static void main(String[] args) {
  
      int[] arr = new int[]{10,5,3,4};
      Arrays.sort(arr);
      System.out.println(rank2(2,arr,0,arr.length-1));
  }
  ```

二分法的缺陷：

必须是有序的，很难保证数组是有序的，虽然可以在构建数组的时候进行排序，这就要考虑另一个瓶颈了：它必须是数组，数组读取效率高，但是插入和删除某个元素的效率比较低，因而构建有序数组比较低效

解决这些缺陷更好的方法是使用`二叉查找树`，最好是`自平衡二叉查找树`