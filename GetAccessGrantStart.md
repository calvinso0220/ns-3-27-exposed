This file explains how ns-3 implements the behavior of 802.11 DCF regarding the random backoff mechanism. The main function is <code>DcfManager::GetAccessGrantStart</code>.

```cpp
Time
DcfManager::GetAccessGrantStart (void) const
{
  NS_LOG_FUNCTION (this);
  Time rxAccessStart;
  if (!m_rxing)
    {
      rxAccessStart = m_lastRxEnd + m_sifs;
      if (!m_lastRxReceivedOk)
        {
          rxAccessStart += m_eifsNoDifs;
        }
    }
  else
    {
      rxAccessStart = m_lastRxStart + m_lastRxDuration + m_sifs;
    }
  Time busyAccessStart = m_lastBusyStart + m_lastBusyDuration + m_sifs;
  Time txAccessStart = m_lastTxStart + m_lastTxDuration + m_sifs;
  Time navAccessStart = m_lastNavStart + m_lastNavDuration + m_sifs;
  Time ackTimeoutAccessStart = m_lastAckTimeoutEnd + m_sifs;
  Time ctsTimeoutAccessStart = m_lastCtsTimeoutEnd + m_sifs;
  Time switchingAccessStart = m_lastSwitchingStart + m_lastSwitchingDuration + m_sifs;
  Time accessGrantedStart = MostRecent (rxAccessStart,
                                        busyAccessStart,
                                        txAccessStart,
                                        navAccessStart,
                                        ackTimeoutAccessStart,
                                        ctsTimeoutAccessStart,
                                        switchingAccessStart
                                        );
  NS_LOG_INFO ("access grant start=" << accessGrantedStart <<
               ", rx access start=" << rxAccessStart <<
               ", busy access start=" << busyAccessStart <<
               ", tx access start=" << txAccessStart <<
               ", nav access start=" << navAccessStart);
  return accessGrantedStart;
}
```

This function is called from two places:

1. <code>DcfManager::IsWithinAifs</code>
2. <code>DcfManager::GetBackoffStartFor</code>

The function <code>IsWithinAifs</code> is to check whether the node is still within the duration of AIFS (DIFS) after the channel becomes idle.

The function <code>GetBackoffStartFor</code> is the important function. This function returns the time *when the backoff has started*.



The <code>GetAccessGrantStart</code> function first checks if the node is currently receiving a packet. The Time variable <code>rxAccessStart</code> is set depending on this state (<code>m_rxing</code>). 

1. If the node is not receiving,

- If last reception was a success, 

<code> rxAccessStart = m_lastRxEnd + m_sifs </code>

- If last reception was a failure,

<code> rxAccessStart = m_lastRxEnd + m_sifs + m_eifsNoDifs </code>

2. If the node is receiving,

<code> rxAccessStart = m_lastRxStart + m_lastRxDuration + m_sifs


The variable <code>rxAccessStart</code> indicates the time when the last *packet reception ended or will end*. Here, three variables are used.

1. <code>m_lastRxStart</code>
2. <code>m_lastRxEnd</code>
3. <code>m_lastRxDuration</code>


<code>m_lastRxStart</code> is set in the function <code>NotifyRxStartNow</code>. <code>m_lastRxEnd</code> is set in the function <code>NotifyRxEndOkNow</code> and <code>NotifyRxEndErrorNow</code>. <code>m_lastRxDuration</code> is set in <code>NotifyRxStartNow</code>. 

If the packet reception is finished abruptly, both <code>m_lastRxEnd</code> and <code>m_lastRxDuration</code> should be updated. In <code>NotifyTxStartNow</code>, these two variables are updated if the node is currently receiving a packet (<code>m_rxing == true</code>). Also, in <code>NotifySwitchingStartNow</code>, the variables are updated if the node has to stop receiving the packet in order to perform channel switching.



