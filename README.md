# DijikesTestCode
Python 3.7 code, demonstrate solution to 1971 Dijkstra backtrack 0,1,2 sequences generator:

*Use nondeterministic choice points to systematically search possible paths to the next sequence meeting the 
specifications of the problem, i.e., to generate in alphabetical order non-empty sequences of the digits (0,1,2) without non-empty
element-wise equal adjoining subsequences until a sequence of arbitrary length is obtained.*

See pg 85 "Notes on Structured Programming" by Prof.dr. Edsger W. Dijkstra, 1970, Technological University Eindhoven, Netherlands, Department of Mathematics, Report 70-WSK-03). We programmed his problem (generating custom digit sequences).

See 10.1.2 Itertools Recipes in "Python Library Reference Release 3.7.6," pg 343 grouper in particular.

The ultimate flow of the program is (following Dijkstra pg 86 loosely)
<pre>
set list of list sequences empty
set length of last verified proposal to zero

while length of last verified proposal is less than the maximum number desired

  extend the proposal sequence with a zero digit (function extend_proposal)
  
  while proposal NOT good 
         (function test_for_repeats of digit patterns from 2 to len/2 of proposal)
         
    increase_proposal: 
     (bump rightmost digit, e.g., 0 to 1, 1 to 2; if proposal already ended in 2's,
          delete the 2's, bump new final)
          
  print/store verified new proposal 
  length = length of verified proposal
      (and begin the loop again with this good proposal, extend and test, 
          if the while length < 100 test passes)  
</pre>

We have **several simplifying facts available** for implementing the "is good"function that validates a proposed sequence:
- Every previous proposal has already been validated, i.e., there is no need to check for duplicate pairs to left from the rightmost two digits.
- There is no need to slide to left over consecutive bits the expanding test window because the adjacent repeated subsequences could only have emerged when the last modification of the final rightmost digit occurred. We need only compare, e.g., first two to adjacent two to left, first three to adjacent three to left, first four to adjacent four to left, etc. 
- The largest pattern comparison, in terms of the number of included digits beginning from rightmost to left, is simply the length of the proposed sequence divided by two if the target has an even number of digits. For an odd number of digits in the proposal, the maximum pattern length is "floor (length of proposal)/2 ) (where floor is the *floor* function nearest integer on low side of a float result). This, of course, works for even number of digits as well so may use the same code with no test for odd or even.

As Dijkstra notes on pg 84 of the cited paper:
>Each solutions (apart from the first one) is an extension (by one digit) of an earlier solution and the algorithm is therefore a straightforward backtracking one.

Hidden within the simplicity of our implementation, which relies on fast higher-order Python routines to chunk our lists, we are really testing nondeterministic choice points to systematically search possible paths to the next sequence meeting the specifications of the problem (generate in alphabetical order non-empty sequences of the digits (0,1,2) without non-empty element-wise equal adjoining subsequences until a sequence of arbitrary length is obtained). Once a proposed sequence is validated, it is extended by a 0 (zero) digit and that result is tested. If that results in a violation of the no-repeat constraint, the final digit is incremented and the test of that path made. If that results in a violation, then the final digit is again incremented, but if it was already the greatest available digit, 2, then we in effect backtrack, deleting that 2 and any prior 2's directly adjoining. We discovered that the path from an 86 digit sequence resulted in a double-two repeat at the end of the sequence, which sent our original evaluation code into an infinite cycle from 87 digits back to 84 digits in length and back until we modified our code to delete multiple repeated 2's at the end, as Dijkstra apparently intended (pg 85):
>If the trial sequence is no good, we perform on it the operation "increase" to get the next trial sequence, i.e., final digits = 2 are removed and then the last remaining digit is increased by 1. (The operations "extend with zero" and "increase" guarantee that trial sequences are generated in alphabetical order..

He also advised that because of the difficulty in analyzing correctness of programs, we should restrict ourselves to 
>simple structures whenever possible and [to] avoid in all intellectual modesty "clever constructions" like the plague.

I'm not sure we have followed that advice here, since, as we already noted, we made use of Python 3.7.4 iterator tools that are fast and memory efficient by themselves or in combination (to quote Guido van Rossum and the Python development team) to simplify our coding effort, if not the code in and of itself. We also made use of the simplifying assumptions we found, making our code less general, i.e., not really a general purpose non-deterministic evaluator, but simplifying our search strategy. 

We initially thought possibly Dijkstra's command of English was not perfect, i.e., that he intended to say "if the increase operation finds a 2, that digit is deleted from the trial sequence and the previous digit, now final digit, is incremented." I would have stated the actual requirement as
>On entry to the increase operation, if the final (rightmost) digit is a 2 (prior to any increment), then remove that digit, decreasing the length of the sequence. If the new final digit is also a 2, remove it as well and decrease the sequence in length again. Repeat until a non-2 final digit is present. Then increment that digit, i.e., from 0 to 1 or from 1 to 2, and return to the primary program sequence the new trial sequence.

This operation is a backtrack (depth-first search or chronological backtrack), reverting to an earlier subsequence and exploring the path forward from that point. Because Dijsktra told us that a sequence of length 100 satisfying the constraints stated does exist, we do not test for the possibility that there is no successful path forward. That is normally a possibility in a non-deterministic evaluation because in general it is impossible to determine, given a description of an arbitrary computer program and an input, whether the program will complete, i.e., halt, or whether it run forever. I do not know specifically whether our specific task here is not computable in that sense, all I can assert is that Dijsktra said it was possible and we verified that (by non-determinist evaluation). 

Dijkstra (on pg 84) provided the first 8 sequences satisfying the problem constraints. We used that as initial target to verify accuracy, though we did discover, as we mentioned above, a bug in our code that only surfaced at a sequence length of 87 digits (a bug which we repaired as discussed above). Our present code generates a 100-digit sequence (and all those up to that length, though did not include code to store those and printing might flood your terminal screen) in less than 8 ms running on a 2.5 GHz 4-core 64-bit Linux machine with 4 GB RAM. As far as reliability on longer sequences, we believe our code probably continues to be accurate, but as Dijkstra pointed out in his 1971 paper, 
>Program testing can be used to show the presence of bugs, but never to show their absence.

With the code copied into `ipython` and then saved as file *202110171230DijkstraOGAS* (download above text file), was able to execute the following after opening a terminal in the containing directory (we set code for 8 digits only, but runs 100 or more)
<pre>
dalton@dalton-Precision-3541:$ ipython 202110171230DijkstraOGAS
['0']
['0', '1']
['0', '1', '0']
['0', '1', '0', '2']
['0', '1', '0', '2', '0']
['0', '1', '0', '2', '0', '1']
['0', '1', '0', '2', '0', '1', '0']
['0', '1', '0', '2', '0', '1', '2']
['0', '1', '0', '2', '0', '1', '2', '0']
dalton@dalton-Precision-3541:$ pwd
/home/dalton/JUP_NBs/general
</pre>

The code in the file *202110171230DijkstraOGAS*:
``` python
import itertools as it
def take(n, iterable):
    "Return first n items of the iterable as a list"
    return list(it.islice(iterable, n))
    
def grouper(iterable, n, fillvalue='F'):
    "Collect data into fixed-length chunks or blocks"
    # grouper('ABCDEFG', 3, 'x') --> ABC DEF Gxx"
    args = [iter(iterable)] * n
    return it.zip_longest(*args, fillvalue=fillvalue)
    
# append a 0 digit to submitted list (do this when the target proposal has been judged "good")
def extend_proposal(target):
    """append a 0 digit to submitted list, modify the list in place"""
    return( target.append('0') )
    
def proposal_good(target):
    """return TRUE if the proposed list sequence (arg target) contains no duplicates"""
    # return if target empty or if only a single digit; length_target variable is used below also
    length_target = len(target)
    if (length_target == 1):
        result_flag = True # single digit cannot be a duplicate pair, so gets a pass
        return(result_flag)
    if (length_target < 1):
        result_flag = False # empty proposal is not a good proposal
        return(result_flag)
    # check rightmost two digits first (no other duplicate pairs should exist elsewhere)
    index_next_to_last = length_target - 2
    rightmost_two_slice = target[index_next_to_last:length_target]
    if (rightmost_two_slice.count('0') > 1):
        result_flag = False # proposal NOT GOOD if pair of zeros on end
        return(result_flag)
    elif (rightmost_two_slice.count('1') > 1):
        result_flag = False # proposal NOT GOOD if pair of ones on end
        return(result_flag)
    elif (rightmost_two_slice.count('2') > 1):
        result_flag = False # proposal NOT GOOD if pair of twos on end
        return(result_flag)
    
    # if we get this far we have no repeat in final right two digits, so return good proposal if len is 2
    if length_target == 2:
        result_flag = True
        return(result_flag)
    
    # if we end up here, target proposal is longer than 2 digits and rightmost pair is not a dup
    # so must scan target chunks from right to left in expanding chunk size from 2, 3, 4 ...max size
    # all base chunks begin at rightmost digit and compare to next adjacent chunk only
    # maximum chunk size is floor( (target length)/2 )
    max_chunk_size = length_target // 2
    chunk_size = 2
    result_flag = True # assume the best and attempt to prove not true in while loop below
    # must use reversed copy because grouper fills odd length from right with F
    reversed_copy = target.copy()
    reversed_copy.reverse()
    while ( chunk_size <= max_chunk_size ):
        reversed_proposal_chunked = list( grouper( reversed_copy, chunk_size, fillvalue='F' ) )
        # the take function conveniently (because reversed list now r to l) takes chunks from left to right
        # do not need to test more than first two chunks from left in reversed proposal
        if ( list(take(2, reversed_proposal_chunked)[0]) == list(take(2, reversed_proposal_chunked)[1]) ):
            result_flag = False # function tests if GOOD proposal, if has dup pattern is BAD!
            break
        # current chunks tested NOT EQUAL above, so expand chunk size and test again
        chunk_size += 1
        
    # if arrived here, then either "breaked" out of loop with a dup pattern, or fell through no dups
    return(result_flag)
    
def increase_proposal(target):
    """bump rightmost digit unless already a 2; otherwise delete the 2 first then bump next to last"""
    length_of_target = len(target)
    index_of_first_item_on_right = length_of_target - 1
    # program flow assures no 0 length targets submitted, otherwise test for that possibility here
    # if come in with rightmost digits equal to "2", delete them then below increment
    while True:
        if (target[index_of_first_item_on_right] == '2'):
            del target[index_of_first_item_on_right]
            length_of_target = len(target)
            index_of_first_item_on_right = length_of_target - 1
        else:
            break
       
    # be safe and do not assume anything about the numeric nature of the digit characters in the list
    if (target[index_of_first_item_on_right] == '0'):
        target[index_of_first_item_on_right] = '1'
    elif (target[index_of_first_item_on_right] == '1'):
        target[index_of_first_item_on_right] = '2'
    return
# the calling program demands that increase_proposal modifies the argument target in place, i.e.
# that no updated list return is required, that the calling program's target is thereby modified
# it having been previously declared globally outside any particular scope
      
proposal_10161207 = list('')
number_of_digits_in_sequence = len(proposal_10161207)
      
# it should be anticipated that number_of_digits_in_sequence may increase and decrease within code below
# say max sequence length desired (comment out print within while loop if more than 8 or so to prevent flood)
# and uncomment the final two prints to see the final sequence of the run and the length of the collection
      
max_seq_len_desired = 8
# proposal_10161207 is a global and is assumed to be modified in place below by called subroutines (functions)
      
# main program loop      
while number_of_digits_in_sequence < max_seq_len_desired:
    # append a "0" digit to last proposal validated (or first empty list if now beginning)
    extend_proposal(proposal_10161207)
    # if the current proposed sequence contains a duplicate pair or pattern r to l, then bump r digit
    # (the increase_proposal function can also delete terminating "2"s and bump the remaining digit)
    
    while ( not proposal_good(proposal_10161207) ):
        increase_proposal(proposal_10161207)

    # if arrive here, then have a good sequence, so print it (or store to print later)
    # uncomment the following print if doing a reasonably small number of digits, e.g., 20
    print("{0}".format(proposal_10161207))
    # update the length of the currently validated sequence
    number_of_digits_in_sequence = len(proposal_10161207)

# if asking for more than 20 digits or so, comment out the print line within the while loop above
# and just print the final length sequence after the while loop below, with the length
"""
print("{0}".format(proposal_10161207))
print("{0}".format( len(proposal_10161207)  ))
""";
```
      
