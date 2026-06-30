# BMW Lab Meeting Guideline

- [BMW Lab Meeting Guideline](#bmw-lab-meeting-guideline)
  - [Meeting Preparation](#meeting-preparation)
    - [**Pre-Meeting Setup (15 minutes before)**](#pre-meeting-setup-15-minutes-before)
    - [**During Meeting Protocol**](#during-meeting-protocol)
    - [**Post-Meeting Documentation**](#post-meeting-documentation)
  - [Weekly Project Discussion](#weekly-project-discussion)
    - [**Meeting Administrator Responsibilities**](#meeting-administrator-responsibilities)
  - [Thesis Discussion](#thesis-discussion)

## Meeting Preparation

> [!IMPORTANT]
> **Purpose**: Create accurate meeting transcriptions and generate comprehensive action items & discussion summaries using AI-powered analysis.

### **Pre-Meeting Setup (15 minutes before)**

1. **Recording Equipment Setup**:
   - Install and configure [OBS Studio](https://obsproject.com/download) or use Microsoft Teams recording
   - Test audio/video capture to ensure quality recording
   - Position camera to capture both presenter and presentation materials

2. **Audio Configuration**:
   - Check and adjust microphone gain levels (test with sample recording)
   - Ensure all participants can be clearly heard
   - Use external microphone if built-in audio quality is poor

3. **Screen Capture Setup**:
   - Configure screen capture to show presentation materials clearly
   - Test screen sharing functionality
   - Ensure all technical diagrams and slides are visible

### **During Meeting Protocol**

4. **Recording Management**:

   **Option A: Local Recording** (Recommended for stability)
   - Record directly to local storage
   - Manually upload to YouTube after meeting
   - Lower CPU usage, better performance

   **Option B: Live Streaming** (For real-time sharing)
   - Stream directly to YouTube Live
   - Higher CPU usage, requires stable internet
   - Immediate availability for remote participants

5. **Action Item Documentation**:
   - **Clearly state participant names** when assigning tasks
   - Use format: "*[Name], please [specific action] by [deadline]*"
   - Repeat action items for clarity and transcription accuracy
   - Confirm understanding before moving to next topic

### **Post-Meeting Documentation**

6. **Transcription Processing**:
   - Wait for YouTube to generate automatic transcript (typically 15-30 minutes)
   - Download transcript in text format
   - Review for accuracy and edit obvious errors

7. **AI-Powered Meeting Minutes Generation**:

   **LLM Prompting Template:**

```terminal
Generate meeting minutes from `<source name>`, focusing on the following topics:

Format the output using the specified GitHub markdown template. Ensure all discussion points are detailed, comprehensive, and categorized under their respective topics. Identify and list all actionable items clearly with checkboxes.

Meeting Context:

• Date: (Get from the file name)

• Time: 

• Attendees: 

Desired GitHub Markdown Format:

\```
## yyyy/mm/dd-hh.mm

- *Reviewed by*: <GitHub username>   <!-- fill in to certify you verified these LLM-generated minutes -->
- *Recording*: <recording link>
- *Attendees*:
- *Topic*:
- *Summary*:

- *Discussion Points*
  - <point-1>
  - <point-n>

- *Action Items*:
  - <attendee-n>:
    - [ ] <action-item-1>
    - [ ] <action-item-n>
\```

---
MEETING TRANSCRIPTION:
# Copy the meeting text transcription bellow:
```

## Weekly Project Discussion

> [!NOTE]
> **Meeting Link**: [Teams Meeting Room](https://teams.microsoft.com/l/meetup-join/19%3ameeting_MjMyMGI3ZjgtNDVjZC00MDc0LTk4NDMtYTc5ZDBhMTQ3NjNm%40thread.v2/0?context=%7b%22Tid%22%3a%22498ea0bc-cc9c-4a4b-bb98-6315bd49503d%22%2c%22Oid%22%3a%22877d059f-5398-4fb8-877d-8ef7cc6df736%22%7d)

### **Meeting Administrator Responsibilities**

**Pre-Meeting (1 week in advance):**

1. **Room & Resource Booking**:
   - Reserve meeting room with appropriate capacity
   - Ensure room has necessary A/V equipment
   - Confirm internet connectivity and backup options
   - Book equipment (projector, speakers, microphones) if needed

2. **Participant Coordination**:
   - Send calendar invitations with agenda
   - Confirm attendance from all required participants
   - Share pre-meeting materials (presentations, reports)
   - Set up meeting link for remote participants

**Day of Meeting (30 minutes before):**

3. **Technical Setup**:
   - Arrive 30 minutes early for equipment testing
   - Set up laptop, projector, and audio system
   - Test screen sharing and recording functionality
   - Verify internet connection and backup solutions
   - Prepare recording equipment (OBS or Teams recording)

4. **Meeting Environment**:
   - Arrange seating for optimal discussion
   - Test microphone levels for all participants
   - Ensure proper lighting for video recording
   - Prepare backup power sources if needed

**During Meeting:**

5. **Meeting Facilitation**:
   - Start recording at meeting beginning
   - Manage time allocation for each agenda item
   - Ensure all participants can contribute effectively
   - Maintain focus on agenda and objectives
   - Document key decisions and action items in real-time

6. **Communication Management**:
   - Moderate discussions to stay on track
   - Ensure remote participants can engage effectively
   - Clarify technical concepts when needed
   - Confirm understanding of assigned tasks

**Post-Meeting (within 24 hours):**

7. **Documentation & Follow-up**:
   - Generate meeting minutes using LLM prompting template (see [Meeting Preparation](#meeting-preparation))
   - Update [Trello Meeting Minutes](https://trello.com/c/QdkJZ1Et) with action items
   - Distribute meeting minutes to all participants
   - Schedule follow-up meetings if required
   - Archive recording and transcription

## Thesis Discussion

For the thesis discussion, each student will choose a sub-group. The sub-group leader will report the result with Prof. Ray.
Sub-group discussion with Professor can be done on demand.

**Requirements:**

1. **Daily Progress Tracking**: Update your Trello board daily with project progress (minimum 6-month historical record required).
2. **Meeting Scheduling**: Confirm weekly meeting schedule with Prof. Ray via [Google Calendar](https://calendar.google.com/calendar/u/0?cid=bW4zbGl0ZzVhdm5saWx0M3Uwb2Nrb3ZzbDRAZ3JvdXAuY2FsZW5kYXIuZ29vZ2xlLmNvbQ) at least 24 hours in advance.
3. **Meeting Presentation Preparation**: Prepare comprehensive thesis plan slides for discussion, including all required technical diagrams:
   
   **Required Diagrams:**
   1. **Use Case Diagram** - Clearly specify all features and functionalities in your project
   2. **System Architecture (Component Diagrams)** - Document tools, frameworks, and technologies used to build your system
   3. **Message Sequence Chart (MSC)** - Illustrate component interactions and data flow for each use case
   4. **Flowchart** - Describe the logic and decision points of your use case implementation:
      1. Show the high-level concept for overall systems
      2. Detailed logic for each algorithm/feature
   5. **Class Diagram** - Show data structures (parameters), functions, and class relationships/interactions

4. **Graduation Timeline & Target Setting**:
   **Choose your graduation timeline:**
   - **Next Semester Graduation** → Establish weekly milestones and deliverables until oral examination
   - **One-Year Graduation Plan** → Define monthly targets and major milestones until oral examination

> [!NOTE]
> All targets must include specific deliverables, deadlines, and success criteria for effective progress tracking.
