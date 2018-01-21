# kickstarter
import pandas as pd
from pandas import Series, DataFrame
from scipy.stats import ttest_ind
%pylab inline
plot_df = pd.concat([kickstarter['launched_at_weekday'].value_counts(),
                     kickstarter['deadline_weekday'].value_counts(),
                    kickstarter['created_at_weekday'].value_counts()], axis=1)
plot_df.index = ["(5) Friday","(1) Monday","(6) Saturday","(7) Sunday","(4) Thursday","(2) Tuesday","(3) Wednesday"]
plot_df = plot_df.sort_index()
plot_df.plot(kind='line', figsize=(10,5), title = "Distribution of Campaigns over Weekdays")
plot_df = pd.concat([kickstarter['deadline_hr'].value_counts(),
                     kickstarter['created_at_hr'].value_counts(),
                    kickstarter['launched_at_hr'].value_counts()], axis=1)
plot_df = plot_df.sort_index()
plot_df.plot(kind='line', figsize=(10,5), title = "Distribution of Campaigns over UTC Day by hour")
pd.qcut(kickstarter['goal'], 10).value_counts().sort_index().plot(kind='bar')
fig, axes = plt.subplots(nrows=5, ncols=2)
fig.subplots_adjust(hspace=1, wspace = 1)

kickstarter['country'].value_counts().plot(kind = 'bar',ax=axes[0,0], title = 'Country Distribution', figsize=(15,15))
kickstarter['currency'].value_counts().plot(kind = 'bar',ax=axes[0,1], title = 'Currency Distribution', figsize=(15,15))
kickstarter['category'].value_counts().plot(kind = 'bar',ax=axes[1,0], title = 'Category Distribution', figsize=(15,15))
kickstarter['deadline_weekday'].value_counts().plot(kind = 'bar',ax=axes[1,1], title = 'Deadline Weekday Distribution', figsize=(15,15))
kickstarter['created_at_weekday'].value_counts().plot(kind = 'bar',ax=axes[2,0], title = 'Created At Weekday Distribution', figsize=(15,15))
kickstarter['launched_at_weekday'].value_counts().plot(kind = 'bar',ax=axes[2,1], title = 'Launched At Weekday Distribution', figsize=(15,15))
kickstarter['deadline_hr'].value_counts().sort_index().plot(kind = 'bar',ax=axes[3,0], title = 'Deadline Hour Distribution', figsize=(15,15))
kickstarter['created_at_hr'].value_counts().sort_index().plot(kind = 'bar',ax=axes[3,1], title = 'Created at Hour Distribution', figsize=(15,15))
kickstarter['launched_at_hr'].value_counts().sort_index().plot(kind = 'bar',ax=axes[4,0], title = 'Launched at Hour Distribution', figsize=(15,15))
kickstarter['state'].value_counts().plot(kind = 'bar',ax=axes[4,1], title = 'State Distribution', figsize=(15,15))
def extract_days(val):
    try:
        l = val.split(" ")
        return int(l[0])
    except:
        return np.nan

kickstarter['create_to_launch_days'] = kickstarter['create_to_launch'].map(extract_days)
kickstarter['launch_to_deadline_days'] = kickstarter['launch_to_deadline'].map(extract_days)
kickstarter['launch_to_state_change_days'] = kickstarter['launch_to_state_change'].map(extract_days)
kickstarter.head(1)

def return_week_bins(val):
    if val < 7:
        return "< 1 week"
    elif val < 14:
        return "1-2 weeks"
    elif val < 28:
        return "2-4 weeks"
    elif val < 35:
        return "4-5 weeks"
    elif val < 42:
        return "5-6 weeks"
    elif val < 56:
        return "6-8 weeks"
    else:
        return "8+ weeks"

successful_state_series = kickstarter[kickstarter['state'] == "successful"]['launch_to_deadline_days']
successful_state_series.map(return_week_bins).value_counts().plot(kind='bar')

def return_date_bins(val):
    if val < 1:
        return "0-1 day"
    elif val < 2:
        return "01-2 days"
    elif val < 7:
        return "02-7 days"
    elif val < 21:
        return "07-21 days"
    elif val < 42:
        return "21-42 days"
    elif val < 63:
        return "42-63 days"
    else:
        return "63+ days"

create_launch_series = kickstarter['create_to_launch_days']
create_launch_series.map(return_date_bins).value_counts().plot(kind='bar')

successful_kickstarter = kickstarter[kickstarter['state'] == "successful"]
failed_kickstarter = kickstarter[kickstarter['state'] == "failed"]
cancelled_kickstarter = kickstarter[kickstarter['state'] == "canceled"]
print len(successful_kickstarter)," successful campaigns"
print len(failed_kickstarter)," failed campaigns"
print len(cancelled_kickstarter)," cancelled campaigns"

def get_perc(val, col):
    return (val / compare_name_len_clean[col]['total'])*100

compare_name_len_clean['all_perc'] = compare_name_len_clean['all'].apply(get_perc, args=('all',))
compare_name_len_clean['success_perc'] = compare_name_len_clean['successful'].apply(get_perc, args=('successful',))
compare_name_len_clean['failed_perc'] = compare_name_len_clean['failed'].apply(get_perc, args=('failed',))
compare_name_len_clean['cancelled_perc'] = compare_name_len_clean['cancelled'].apply(get_perc, args=('cancelled',))

compare_name_len_clean[['success_perc','failed_perc']][:14].sort_index().plot(kind='bar', figsize=(10,5), title = "Distribution of length of name for successful and failed campaigns")

print ttest_ind(failed_kickstarter[n], successful_kickstarter[n])
print "Failed mean: ", failed_kickstarter[n].mean()
print "Successful mean: ", successful_kickstarter[n].mean()

def get_goal_buckets(val):
    if val < 1001:
        return "0-1000"
    elif val < 5001:
        return "1001-5000"
    elif val < 10001:
        return "5001-10000"
    elif val < 50001:
        return "10001-50000"
    else:
        return "50001+"

n = 'goal'
compare_goal = pd.concat([kickstarter[n].apply(get_goal_buckets).value_counts(),successful_kickstarter[n].apply(get_goal_buckets).value_counts(),failed_kickstarter[n].apply(get_goal_buckets).value_counts(),cancelled_kickstarter[n].apply(get_goal_buckets).value_counts()], axis=1)
compare_goal.columns = ['all','successful','failed','cancelled']
compare_goal.loc['total'] = compare_goal.sum(axis=0)

compare_goal['all_perc'] = compare_goal['all'].apply(get_perc, args=('all',))
compare_goal['success_perc'] = compare_goal['successful'].apply(get_perc, args=('successful',))
compare_goal['failed_perc'] = compare_goal['failed'].apply(get_perc, args=('failed',))
compare_goal['cancelled_perc'] = compare_goal['cancelled'].apply(get_perc, args=('cancelled',))

compare_goal[['success_perc','failed_perc','cancelled_perc']][:5].sort_index().plot(kind='bar', figsize=(10,5), title = "Distribution of goal for successful, failed and cancelled campaigns")

compare_launched_df = pd.concat([successful_kickstarter['launched_at_weekday'].value_counts(), failed_kickstarter['launched_at_weekday'].value_counts()], axis=1)
compare_launched_df.columns = ['successful','failed']
compare_L2D.loc['total'] = compare_C2L.sum(axis=0)

compare_launched_df['success_perc'] = compare_launched_df['successful'].apply(get_perc, args=('successful',))
compare_launched_df['failed_perc'] = compare_launched_df['failed'].apply(get_perc, args=('failed',))
compare_launched_df.index = ["(5) Friday","(1) Monday","(6) Saturday","(7) Sunday","(4) Thursday","(2) Tuesday","(3) Wednesday"]
compare_launched_df = compare_launched_df.sort_index()
# https://data.world/rdowns26/kickstarter-campaigns/workspace/file?filename=Analysis.ipynb
compare_launched_df[['success_perc','failed_perc']].plot(kind='bar', figsize=(10,5), title = "Distribution of weekday launch for successful and failed campaigns")
