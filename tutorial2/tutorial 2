#include <stdio.h>

int binarySearch(int arr[], int low, int high, int target)
{
    if(low > high)
        return -1;

    int mid = (low + high) / 2;

    if(arr[mid] == target)
        return mid;

    if(arr[mid] > target)
        return binarySearch(arr, low, mid - 1, target);

    return binarySearch(arr, mid + 1, high, target);
}

int main()
{
    int n, target, result;

    printf("Enter number of elements: ");
    scanf("%d", &n);

    int arr[n];

    printf("Enter %d sorted elements:\n", n);

    for(int i = 0; i < n; i++)
        scanf("%d", &arr[i]);

    printf("Enter element to search: ");
    scanf("%d", &target);

    result = binarySearch(arr, 0, n - 1, target);

    if(result != -1)
        printf("Element found at index: %d\n", result);
    else
        printf("Element not found\n");

    return 0;
}
