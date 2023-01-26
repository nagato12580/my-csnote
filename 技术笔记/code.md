
[leetcode.27 移除元素](https://leetcode.cn/problems/remove-element/) #双指针 #暴力 ^259fe0
```c++
//双指针法
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int slowly=0;
        for(int fast=0;fast<nums.size();fast++){
            if(nums[fast]!=val){
                nums[slowly++]=nums[fast];
            }
        }
        return slowly;
    }

};
```

[leetcode.977 有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/) #双指针 ^183535
```c++
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        int slow=0,fast=nums.size()-1;
        vector<int> newArray(nums.size(),0); 
        int i=nums.size()-1;
        while(slow<=fast){
            if(nums[slow]*nums[slow]<=nums[fast]*nums[fast]){
                newArray[i--]=nums[fast]*nums[fast];
                --fast;
            }
            else{

                newArray[i--]=nums[slow]*nums[slow];
                ++slow;
            }
        }
        return newArray;
    }
};
```