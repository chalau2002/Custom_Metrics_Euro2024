# Introduction

This project aims to apply football data analysis techniques using event data and 360° data, exploring the potential of the StatsBomb dataset and the 360° layer. Through tactically relevant metrics, it seeks to observe both individual and collective behaviors.

## Data Used

The data comes from the **StatsBomb** platform, including:

- **Events**: actions such as passes, shots, pressures, interceptions, among others, with high granularity.  
- **360° Data**: captures of the positional context at the moment of events, allowing analyses of passing angles, pressure lines, and teammate availability.

## Player Profiles

After defining and computing advanced metrics, the next step is the creation of player profiles based on their performance across multiple matches. The goal is to identify tactical similarities and differentiate between roles performed on the pitch.

## Methodology

To define and compare player profiles, quantitative approaches based on statistical and machine learning principles are applied. These methodologies allow for the identification of similarity patterns between players and the assessment of the quality of the generated segmentation.

## Justification for Game Selection: Turkey vs Georgia (UEFA Euro 2024)

The choice of the Turkey vs Georgia match, played in the context of Euro 2024, is based on two criteria: tactical context diversity and analytical potential.

This match presents a balanced distribution of defensive and offensive actions across different areas of the pitch, with significant involvement of players from various profiles and positions. Such diversity allows the observation of varied tactical behaviors and the application of advanced metrics in a representative and comparable way.

Additionally, it was one of the most contested games of the group stage, with a high density of relevant actions — including pressures, recoveries, and offensive movements — which enriches the sample of events and allows characterization of players with distinct tactical profiles.

Thus, the Turkey vs Georgia game constitutes a solid basis for applying advanced metrics that analyze player impact in different defensive and offensive contexts.

---

## Reasoning for Metric Selection

The selection of metrics in this work aimed to ensure tactical relevance and complementarity between distinct dimensions of the game. The goal was to select indicators that allow observation of meaningful player behaviors in critical moments, both on and off the ball.

It was also important to ensure that at least one metric leveraged the potential of the 360° data, adding spatial context to the analysis. The final choice focused on two metrics that, together, characterize both defensive intensity and decision-making in possession relative to teammate proximity: **Pressures Leading to Recovery (PLR)** and **Passing Lines in Build-Up**.

This combination also aims to capture player involvement and proactivity, both in reaction to losing possession and in participation in collective dynamics with the ball.

Beyond their individual analytical utility, these metrics provide a solid foundation for creating player tactical profiles, enabling segmentation of different styles and roles based on observable and measurable behaviors.

---

## 1. Pressures Leading to Recovery (PLR)

### Description
The **Pressures Leading to Recovery (PLR)** metric measures the proportion of pressures performed by a player that result in their team regaining possession within the next 5 seconds. This approach distinguishes players whose defensive actions directly contribute to collective success in ball recovery.

### Tactical Justification
This metric aligns with modern concepts of coordinated pressing and reaction to possession loss. It highlights players with real defensive impact rather than just action volume. By considering the outcome of the pressure, it introduces a qualitative dimension absent in other metrics.

### Calculation
For each event of type `"Pressure"`:

1. Record the timestamp and player's team.  
2. Search in the next 5 seconds for events such as `"Ball Recovery"`, `"Interception"`, or successful `"Duel"` by the same team.  
3. Only successful recoveries (e.g., without `recovery_failure`) are counted.  
4. The metric is calculated as:

\[
\text{PLR} = \frac{\text{Number of pressures leading to recovery}}{\text{Total pressures performed}}
\]

### Example
If a player applied 20 pressures and their team recovered the ball in 10 of these situations within the next 5 seconds:

\[
\text{PLR} = \frac{10}{20} = 0.5
\]

### Note
While useful, this metric depends on the defined temporal window and does not capture tactically induced recoveries outside this interval. Nevertheless, it provides a solid measure of pressure effectiveness.

<img width="583" height="630" alt="Captura de ecrã 2026-02-13, às 16 32 54" src="https://github.com/user-attachments/assets/bc9a56f9-575f-4bb1-b7ce-f8a8b22354ff" />

## Analysis of Pressures Leading to Recovery (PLR)

Although Georgia was the team that applied the most pressures during the match, PLR values reveal that Turkey achieved higher efficiency in immediate possession recovery (average PLR: Turkey 0.561 vs Georgia 0.529). This contrast suggests that, while the Georgians pressed more often, the Turks managed to do so in a more coordinated and effective manner.

### Offensive Sector
Turkish forwards stood out with very high PLR values, despite performing fewer pressure actions than their opponents.  
Players such as **Arda Güler** (PLR: 0.86), **Yilmaz** (0.7), and **Kenan Yildiz** (0.83) demonstrated a strong ability to recover possession with relatively few actions, reinforcing the idea of a coordinated high press, applied at the right moments with strong collective efficiency. All these players had a similar number of pressures, further supporting this observation.

Georgia also had active forwards in pressing, notably **Georges Mikautadze**, one of the players with the highest volume of pressure actions and a fairly high PLR (0.65) relative to the number of pressures performed.  
Still, the average efficiency of Georgian forwards was lower than that of the Turkish forwards, indicating potentially lower collective coordination or reduced success in executing the press in advanced zones, with **Kvaratskhelia** showing a PLR of 0.4.

### Central Midfield
In central midfield, Georgia remained the team with the highest pressing activity, with players such as **Anzor Mekvabishvili** standing out with a PLR of 0.59. While other midfielders were more active in pressing, their effectiveness was lower.  

This contrasts with Turkish midfielders, who pressed less frequently but more effectively, with **Kaan Ayhan** reaching a PLR of 1.0. **Orkun Kökçü**, however, showed difficulties converting pressures into effective recoveries, registering lower PLR values.

### Contextual Considerations
This scenario can partly be explained by the match context — Georgia spent more time trailing and was forced to press more frequently, especially in mid and defensive zones.  
Such behavior suggests a more reactive approach, where the team had to take higher risks and actively seek to regain possession in order to launch attacks, responding to the need to overturn the result.


## 2. Passing Lines in Build-Up

### Description
The **Passing Lines in Build-Up** metric quantifies how often a player positions themselves as a viable passing option for the ball carrier during carry actions. Using StatsBomb 360° data, it evaluates whether the off-ball player is located within a relevant zone (a 120° cone with an adjustable radius) and not at risk of interception by opponents, reflecting their contribution to supporting the progression of play.

In this analysis, central defenders and goalkeepers are excluded, since by the nature of their positions they are frequently unmarked in deeper areas, which could bias the metric in their favor without reflecting true offensive involvement or intention to progress the ball.

### Tactical Justification
During build-up phases, creating passing lines is essential to maintain offensive fluidity, attract defensive attention, and allow clean exits under pressure. This metric identifies players with good positional awareness and dynamic support, assessing not only proximity but also the relevance of their positioning relative to the ball carrier's context.

### Calculation
For each `"Carry"` event with available freeze-frame data:

1. Determine the direction of the ball carrier's movement.  
2. For each teammate (teammate = True), evaluate:  
   - If positioned within a 120° cone centered on the carry direction;  
   - If within the following distance thresholds:  
     - ≤ 35 meters in the first two-thirds of the pitch  
     - ≤ 25 meters in the final offensive third (x ≥ 80 in StatsBomb coordinates)  
   - If no visible opponent blocks the direct passing lane, considering a defensive influence zone of ~2 meters.  
3. A player is considered a valid passing option if all the above criteria are met.  

The metric for each player is then calculated as:

\[
\text{Passing Line Metric} = \frac{\text{Number of valid passing options}}{\text{Total carry events with freeze-frame}}
\]

### Example
If a player appears in 60 carry events with freeze-frame, and in 15 of them is considered a valid passing option:

\[
\text{Metric} = \frac{15}{60} = 0.25
\]

### Note
Although this metric provides rich insight into offensive involvement, it has some limitations:

- It is based only on carry events with available 360° data, not capturing contexts without ball progression.  
- It assumes the passing line is straight and instantaneous, ignoring dynamic obstacles and subsequent opponent movements.  
- Player visibility depends on the freeze-frame, which may omit valid positions outside the camera view.  
- It may favor players operating in deeper positions (less pressure), hence the need to filter defensive positions in analyses focused on offensive creation.

<img width="776" height="530" alt="Captura de ecrã 2026-02-13, às 16 34 02" src="https://github.com/user-attachments/assets/63b60ce9-50b1-462e-9992-07f1800296da" />

## Illustration of Passing Lines in Build-Up

The image above illustrates the **Passing Lines in Build-Up** metric. The red dot represents the start of a ball carry, with the arrow indicating its direction (in this case, the carry was short, so the arrow is barely visible). The shaded area defines the viable passing zone: a 120° cone with a radius of up to 25 meters, centered on the carry direction.

Blue circles represent teammates visible at that moment, while black circles represent opponents. Only players positioned within this zone and not blocked by opponents are considered valid passing options. These players are connected to the ball carrier with dashed blue lines.

The metric evaluates, for each player, how frequently they appear as a valid passing option during carries, reflecting their availability and involvement in the team's offensive actions.

<img width="626" height="428" alt="Captura de ecrã 2026-02-13, às 16 34 49" src="https://github.com/user-attachments/assets/47f9c062-2c2f-4e09-8f62-c3f7da894d54" />

## Considerations and Data Extraction

### Positional Bias of the Metric
This metric tends to favor deeper-lying players — such as defenders and defensive midfielders — for two main reasons:

1. **Less-marked influence zones**: deeper areas provide more space and time for the player to position themselves as a passing option.  
2. **Spatial criteria of the metric**: the evaluated area is larger in the first two-thirds of the pitch.

Players such as **Mekvabishvili** (CDM, Georgia), **Çalhanoğlu** (RDM, Turkey), and **Giorgi Kochorashvili** (LCM, Georgia) stand out as key contributors during the initial phase of build-up.

### Positional Tendencies
Based on theoretical positions, it could be assumed that Georgia had stronger support on the left flank, while Turkey had it on the right flank, considering players with the highest metric scores:  
- Georgia: **Kochorashvili** (LCM), **Tsitaishvili** (LWB), **Kvaratskhelia** (LCF)  
- Turkey: **Müldür** (RB), **Çalhanoğlu** (RDM), **Arda Güler** (RW)  

However, visual analysis on the Power BI dashboard reveals that this tendency holds for Georgia, while in Turkey it is primarily the right-back (**Müldür**) who contributes significantly on that flank. **Çalhanoğlu** and **Güler**, although nominally associated with the right side, frequently appear as passing options in more central areas, consistent with their role in the team's offensive dynamics.  

Additionally, most passing lines appear in deeper zones for Georgia compared to Turkey, where they emerge in more advanced areas.

### Notable Offensive Players
Although the metric is primarily build-up oriented, two offensive players stand out with strong support contributions in advanced zones:

- **Arda Güler** – Score: 0.60: one of the main offensive references on the flank; demonstrates high positional intelligence.  
- **Khvicha Kvaratskhelia** – Score: 0.57: confirms his role as Georgia’s offensive engine, offering himself in between lines.  

Both players are highly influential in their teams, actively seeking involvement in the game, which reinforces the metric’s validity for offensive contexts despite its limitations.

### Data Extraction: Euro 2024 Group Stage Matches
To ensure fair and balanced analysis among players, only the **group stage of Euro 2024** was considered. This stage provides a solid basis for analysis since all teams play exactly three matches, ensuring a uniform number of opportunities for players to participate and demonstrate performance.

The knockout stage was excluded because not all teams advance, which would compromise comparability of the generated player profiles.

To guarantee statistical reliability, only players who played a **minimum of 90 minutes across the three group-stage matches** were considered. This criterion avoids including players with residual participation whose metrics would not represent their true performance profile.

## Player Profile Analysis Based on Calculated Metrics

In this final phase of the project, the goal is to identify and compare player profiles based on the previously calculated advanced metrics.

The approach follows these steps:

### 1. Determining the Optimal Number of Clusters
The **Elbow Method** and the **Silhouette Score** are used to identify the ideal number of clusters to generate. These techniques evaluate intra-cluster and inter-cluster variability to determine a meaningful segmentation.

### 2. Applying a Clustering Algorithm
The **K-Means** algorithm is applied to segment players into clusters with similar characteristics, using the calculated metrics and the optimal number of clusters determined in the previous step.

### 3. Characterization and Interpretation of Profiles
Each cluster is analyzed in terms of average performance across the considered metrics, with particular attention to tactical context and the potential role of each profile in different game situations.

---

### Determining the Optimal Number of Clusters (Detailed)

The **Elbow Method** is a technique used to find the ideal number of clusters in a clustering algorithm such as K-Means. It is based on the analysis of **inertia** (within-cluster sum of squares), which measures cluster compactness — that is, how close the points are within each group.

As the number of clusters \( k \) increases, inertia tends to decrease because clusters become smaller and better fitted to the data.

However, there is a point at which the additional gain is marginal. This point is known as the **“elbow”**, as the inertia curve tends to bend here — suggesting that increasing the number of clusters beyond this point does not provide significant improvement.

To complement the Elbow Method, the **Silhouette Score** is also used. This score measures internal cluster coherence (proximity between elements within the same group) and external separation (distance to other clusters). Values range from -1 to 1, where higher values indicate well-defined clusters.

By analyzing both the inflection in the inertia curve and the Silhouette Score values simultaneously, it is possible to more robustly identify the optimal number of clusters to use.

<img width="825" height="321" alt="Captura de ecrã 2026-02-13, às 16 36 43" src="https://github.com/user-attachments/assets/32650c61-cf07-43b2-b763-4be7035bb9f9" />

### Optimal Number of Clusters

Based on the Silhouette Score analysis, the optimal \( k \) is 8.  

However, combining insights from the **Elbow Method** and the **Silhouette Score**, it is concluded that the ideal number of clusters is **k = 4**.  

- In the first chart (Elbow Method), a sharp change in the slope occurs between **k = 4** and **k = 5**, suggesting that these values mark the point where additional gains in cluster compactness become marginal — the so-called "elbow."  
- In the second chart (Silhouette Score), although the maximum value occurs at **k = 8**, the score for **k = 4** is very close to this peak, indicating that clusters formed with four groups still maintain good internal coherence and external separation.

Thus, based on the combined analysis of both methods, **k = 4** is selected as the most appropriate choice, representing a solid compromise between segmentation quality and interpretability of the resulting player profiles.

<img width="478" height="366" alt="Captura de ecrã 2026-02-13, às 16 37 42" src="https://github.com/user-attachments/assets/54e60215-a263-4f24-b7cb-06f0dc797cea" />

### Silhouette Score and Profile Interpretation

The obtained **Silhouette Score** was **0.358**, a value which, although not high, is considered reasonable in the context of real-world data with low dimensionality — especially given that the segmentation was based on only two variables.

Despite some overlap between clusters, particularly in the central region of the plot, the groups formed demonstrate a clear and coherent structure. This limitation in separation is explained by the dimensional constraint of the analysis — using only two metrics, while justified, reduces the ability to capture more complex nuances of player behavior. Nonetheless, the results are statistically valid and tactically interpretable.

---

## Characterization and Interpretation of Player Profiles

The analysis of the four clusters allows identification of distinct player profiles based on **offensive contribution** (ability to provide passing lines) and **defensive contribution** (effectiveness in recovery after pressure):

### Cluster 0 (Blue)
- Players with low effectiveness in post-pressure recovery and median values in passing line creation.  
- Possible profile: Players less involved in immediate possession recovery but with variable availability in offensive zones. Could correspond to wingers maintaining wide positions with less between-the-lines presence, or midfielders with lower defensive responsibility but some influence in build-up.

### Cluster 1 (Orange)
- Players with intermediate PLR but low passing line creation.  
- Possible profile: Balanced tactical contributors, not standout in offensive support but relevant in ball recovery. May include defensive or coverage midfielders, or full-backs with more defensive roles, less involved in build-up phases.

### Cluster 2 (Green)
- Players with intermediate PLR but high ability to provide passing lines during build-up.  
- Combine offensive positional intelligence with some defensive effectiveness.  
- Possible profile: Players with high positional awareness and offensive involvement, frequently available as passing options in useful zones. Could include attacking midfielders, inside forwards, or wingers actively participating in offensive circulation and high pressing.

### Cluster 3 (Red)
- Players with high effectiveness in immediate possession recovery and median passing line creation.  
- Strong impact in defensive pressing, with less emphasis on offensive involvement.  
- Possible profile: Players with high defensive pressing impact, frequently recovering the ball but less involved in build-up. Could include defensive midfielders or forwards with high pressing ability.

---

The creation of these profiles provides an objective basis for analyzing individual tactical performance. It enables pattern identification and complements traditional qualitative assessment. Although limited to two metrics, this approach can be expanded in the future with additional variables to increase robustness and granularity of the generated player profiles.

A **Power BI dashboard** was also developed to facilitate the analysis and visualization of the metrics obtained for the Turkey vs Georgia match, providing an interactive and intuitive way to explore player performance and tactical patterns.

<img width="1500" height="809" alt="Captura de ecrã 2026-02-13, às 16 39 14" src="https://github.com/user-attachments/assets/b3994dce-753a-4662-85a8-77be0c99fc64" />

