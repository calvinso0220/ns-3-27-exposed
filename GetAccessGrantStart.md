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

1.1. If last reception was a success, 

<code> rxAccessStart = m_lastRxEnd + m_sifs </code>

1.2. If last reception was a failure,

<code> rxAccessStart = m_lastRxEnd + m_sifs + m_eifsNoDifs </code>

