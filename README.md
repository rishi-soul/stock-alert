if(coveragePlansResponse instanceof LoadedState) {
        if (coveragePlans && coveragePlans.length > 0) {
          if (!coveragePlans.some(p => p.isSelected)) {
            // Determine selected plan by sorting them
            // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort
            coveragePlans.sort((a, b) => {
              // First Criteria: Active over Inactive
              if (a.coverageStatus === CoverageStatus.Active && b.coverageStatus !== CoverageStatus.Active) {
                return -1;
              } else if (a.coverageStatus !== CoverageStatus.Active && b.coverageStatus === CoverageStatus.Active) {
                return 1;
                // Second Criteria: Medical over Dental
              } else if (a.medicalProductPlan && !b.medicalProductPlan) {
                return -1;
              } else if (!a.medicalProductPlan && b.medicalProductPlan) {
                return 1;
              } else {
                // Third Criteria: Subscriber, then Spouse then dependent
                const accessToken = this._authService.myBlueAccessToken();  // need to know information for logged in user
                const subscriberId = accessToken?.claims.subscriberId ?? '';
                const spouseId = accessToken?.claims.spouseId ?? '';
                // Subscriber
                if (a.planMembers.find(pm => pm.relationshipToSubscriber === SubscriberRelationship.Self && pm.memberContractId === subscriberId) &&
                  !b.planMembers.find(pm => pm.relationshipToSubscriber === SubscriberRelationship.Self && pm.memberContractId === subscriberId)) {
                  return -1;
                } else if (b.planMembers.find(pm => pm.relationshipToSubscriber === SubscriberRelationship.Self && pm.memberContractId === subscriberId) &&
                  !a.planMembers.find(pm => pm.relationshipToSubscriber === SubscriberRelationship.Self && pm.memberContractId === subscriberId)) {
                  return 1;
                  // Spouse
                } else if (a.planMembers.find(pm => pm.relationshipToSubscriber === SubscriberRelationship.Spouse && pm.memberContractId === spouseId) &&
                  !b.planMembers.find(pm => pm.relationshipToSubscriber === SubscriberRelationship.Spouse && pm.memberContractId === spouseId)) {
                  return -1;
                } else if (b.planMembers.find(pm => pm.relationshipToSubscriber === SubscriberRelationship.Spouse && pm.memberContractId === spouseId) &&
                  !a.planMembers.find(pm => pm.relationshipToSubscriber === SubscriberRelationship.Spouse && pm.memberContractId === spouseId)) {
                  return 1;
                }
              }
              // Can not decide for both dependents
              return 0;
            });
            // Initialize state data
            coveragePlans[0].isSelected = true;
            this._planDataService.setSelectedCoveragePlan(coveragePlans[0].cardId, coveragePlans[0].planDatePeriod.fromDate).subscribe();
          }
