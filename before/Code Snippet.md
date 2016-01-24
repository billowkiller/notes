1. **在一个二叉树中查找值为x的结点，并打印该结点所有祖先结点的算法，也就是根节点到x的路径。**

	思路1： 利用**后序遍历**的非递归算法，在访问结点时增加一个判断，若该结点的值等于x，则打印栈中保持的路径，再中断算法。
	
	思路2： 采用**前序遍历**的递归算法，在典型的便利算法的参数表中增加了x、path[n]等参数。
	
	    template<class T>
	    int Find_Print(BinTreeNode<T*& BT, T x, T path[], int level, int &count) {
			if(BT != NULL) {
				level++;
				path[level] = BT->data;
				if (BT->data==x) { count=level; return 1; }
				if( Find_Print(BT->left, x, path, level, count) ) return 1;
				return Find_Print(BT->right, x, path, level, count);
			} else 
				return 0;
		}

2. **判断二叉树是不是平衡**

		bool IsBalanced(BinaryTreeNode* pRoot)
		{
		    if(pRoot == NULL)
		        return true;
		 
		    int left = TreeDepth(pRoot->m_pLeft);
		    int right = TreeDepth(pRoot->m_pRight);
		    int diff = left - right;
		    if(diff > 1 || diff < -1)
		        return false;
		 
		    return IsBalanced(pRoot->m_pLeft) && IsBalanced(pRoot->m_pRight);
		}
	
	一个节点会被重复遍历多次，这种思路的时间效率不高。用**后序遍历**的方式遍历二叉树的每一个结点，在遍历到一个结点之前我们已经遍历了它的左右子树。只要在遍历每个结点的时候记录它的深度，我们就可以一边遍历一边判断每个结点是不是平衡的。
	
		bool IsBalanced(BinaryTreeNode* pRoot, int* pDepth)
		{
		    if(pRoot == NULL)
		    {
		        *pDepth = 0;
		        return true;
		    }
		 
		    int left, right;
		    if(IsBalanced(pRoot->m_pLeft, &left)
		        && IsBalanced(pRoot->m_pRight, &right))
		    {
		        int diff = left - right;
		        if(diff <= 1 && diff >= -1)
		        {
		            *pDepth = 1 + (left > right ? left : right);
		            return true;
		        }
		    }
		    return false;
		}
		bool IsBalanced(BinaryTreeNode* pRoot)
		{
		    int depth = 0;
		    return IsBalanced(pRoot, &depth);
		}

3. Reverse Bits

		 // special case: only works for 4 bytes (32 bits).
	
	    uint reverseMask(uint x) 
		{
	    	assert(sizeof(x) == 4);
	    	x= ((x & 0x55555555) << 1) | ((x & 0xAAAAAAAA) >> 1);
	    	x = ((x & 0x33333333) << 2) | ((x & 0xCCCCCCCC) >> 2);
	    	x = ((x & 0x0F0F0F0F) << 4) | ((x & 0xF0F0F0F0) >> 4);
	    	x = ((x & 0x00FF00FF) << 8) | ((x & 0xFF00FF00) >> 8);
	    	x= ((x & 0x0000FFFF) << 16) | ((x & 0xFFFF0000) >> 16);
	    	return x;
	    }

4. Find the k-th Smallest Element in the Union of Two Sorted Arrays

		int findKthSmallest(int A[], int m, int B[], int n, int k) {
		  assert(m >= 0); assert(n >= 0); assert(k > 0); assert(k <= m+n);
		  
		  int i = (int)((double)m / (m+n) * (k-1));
		  int j = (k-1) - i;
		 
		  assert(i >= 0); assert(j >= 0); assert(i <= m); assert(j <= n);
		  // invariant: i + j = k-1
		  // Note: A[-1] = -INF and A[m] = +INF to maintain invariant
		  int Ai_1 = ((i == 0) ? INT_MIN : A[i-1]);
		  int Bj_1 = ((j == 0) ? INT_MIN : B[j-1]);
		  int Ai   = ((i == m) ? INT_MAX : A[i]);
		  int Bj   = ((j == n) ? INT_MAX : B[j]);
		 
		  if (Bj_1 < Ai && Ai < Bj)
		    return Ai;
		  else if (Ai_1 < Bj && Bj < Ai)
		    return Bj;
		 
		  assert((Ai > Bj && Ai_1 > Bj) || 
		         (Ai < Bj && Ai < Bj_1));
		 
		  // if none of the cases above, then it is either:
		  if (Ai < Bj)
		    // exclude Ai and below portion
		    // exclude Bj and above portion
		    return findKthSmallest(A+i+1, m-i-1, B, j, k-i-1);
		  else /* Bj < Ai */
		    // exclude Ai and above portion
		    // exclude Bj and below portion
		    return findKthSmallest(A, i, B+j+1, n-j-1, k-j-1);
		}
